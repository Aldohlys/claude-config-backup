# Test Shiny Application Skill

Testing patterns for Shiny applications (not R packages) in the RTrading system.

## When to Use This Skill

- Testing Shiny applications: RReporting, RJournal, ROrder, RStudies, Tuser apps
- Creating test scripts for new Shiny modules
- Debugging test failures in applications

## CRITICAL: Shiny Apps vs R Packages

**Shiny Applications** (RReporting, RJournal, ROrder, RStudies, Tuser apps):
- ❌ `devtools::test()` does NOT work - these are NOT packages
- ✅ Use standalone test scripts: `test_automated.R`, `test_manual_validation.R`
- ✅ Run with: `Rscript path/to/test_script.R`
- ✅ Tests can be at application root or in subdirectories

**R Packages** (Tbasics, Tdata, Tlogger):
- ✅ Use `devtools::test()` - works with testthat framework
- ✅ Tests located in `tests/testthat/` directory

**Common Mistake**:
```r
# ❌ WRONG - RReporting is an application, not a package
cd RReporting && devtools::test()

# ✅ CORRECT - Use standalone test script
cd RReporting && Rscript test_automated.R
```

## Three Testing Patterns for Shiny Apps

### 1. Standalone Validation Tests

Test business logic directly without Shiny framework mocking.

**When to use**: Testing utility functions, data transformations, calculations

**Example Structure**:
```r
# test_automated.R
library(testthat)

# Load your modules/functions
box::use(
  utils/calculations[compute_pnl, compute_greeks]
)

test_that("PnL calculation is correct", {
  # Test pure functions without Shiny reactivity
  result <- compute_pnl(
    positions = data.frame(symbol = "AAPL", pos = 100, price = 150),
    initial_cost = 14000
  )

  expect_equal(result$unrealized_pnl, 1000)
})

# Run all tests
test_dir(".", pattern = "^test_.*\\.R$")
```

### 2. Integrated Tests with Mocked Reactives

Test module interactions with Shiny reactive values mocked.

**When to use**: Testing Shiny modules, reactive logic, UI/server interactions

**Example Structure**:
```r
# test_module_integration.R
library(shiny)
library(testthat)

box::use(
  analysis/view/an_lineUI
)

test_that("Module returns expected reactive values", {
  # Use testServer to test module server logic
  testServer(
    an_lineUI$server,
    args = list(
      portfolio = reactive(test_portfolio_data)
    ),
    {
      # Test reactive values
      expect_true(!is.null(session$returned$data()))
      expect_equal(nrow(session$returned$data()), 5)

      # Test reactive expressions
      session$setInputs(filter_symbol = "AAPL")
      expect_true("AAPL" %in% session$returned$data()$symbol)
    }
  )
})
```

### 3. Manual Testing Scripts

Interactive testing scripts for development and validation.

**When to use**: Manual verification, visual testing, debugging

**Example Structure**:
```r
# test_manual_validation.R
library(shiny)

box::use(
  analysis/view/an_lineUI
)

# Create minimal app for manual testing
ui <- fluidPage(
  titlePanel("Manual Test: Analysis Module"),
  an_lineUI$ui("test")
)

server <- function(input, output, session) {
  # Load test data
  test_portfolio <- reactive({
    # ... load or generate test data
  })

  # Call module
  result <- an_lineUI$server("test", portfolio = test_portfolio)

  # Print outputs to console for verification
  observe({
    cat("Module data:\n")
    print(result$data())
  })
}

# Run app
shinyApp(ui, server)
```

## Test File Organization

### Recommended Structure

```
ApplicationRoot/
├── test_automated.R         # Main test suite (run via Rscript)
├── test_manual_validation.R # Interactive testing
├── tests/
│   ├── test_calculations.R  # Business logic tests
│   ├── test_modules.R       # Module integration tests
│   └── test_data.R          # Data processing tests
└── test_data/               # Test fixtures
    ├── sample_portfolio.rds
    └── mock_trades.csv
```

## Running Tests

### Automated Tests

```bash
# From application directory
Rscript test_automated.R

# Specific test file
Rscript tests/test_calculations.R

# With verbose output
Rscript -e "testthat::test_dir('.', reporter = 'progress')"
```

### Manual Tests

```bash
# Run interactive Shiny app for manual validation
Rscript test_manual_validation.R
```

## Test Coverage Best Practices

### What to Test

✅ **Business Logic**:
- Calculations (PnL, Greeks, margins)
- Data transformations
- Portfolio operations

✅ **Module Return Values**:
- Reactive data structures
- Function calls
- State management

✅ **Error Handling**:
- Invalid inputs
- Missing data
- Edge cases

❌ **Don't Test**:
- UI rendering (use manual tests instead)
- External APIs (mock them)
- Database connections (use test database or mocks)

### Syntax Validation

**IMPORTANT**: DO NOT run standalone syntax validation:

```r
# ❌ WRONG - Wastes tokens, duplicates existing checks
tryCatch(parse("file.R"), error = function(e) ...)

# ✅ CORRECT - Syntax validation is automatic in test suites
# Just run: Rscript test_automated.R
```

Syntax validation is handled automatically by:
- Test suites (`test_automated.R` includes syntax checks)
- The `/build` command
- R's normal loading process

## Test Data Management

### Using Test Fixtures

```r
# Load test data
test_portfolio <- readRDS("test_data/sample_portfolio.rds")

# Or generate programmatically
create_test_portfolio <- function() {
  data.frame(
    symbol = c("AAPL", "MSFT", "GOOGL"),
    pos = c(100, 50, 25),
    mktPrice = c(150, 300, 2800),
    avgCost = c(140, 290, 2700)
  )
}
```

### Handling IBKR vs Gonet Differences

When testing code that handles both account types:

```r
# Test with IBKR data (full columns)
test_that("Works with IBKR portfolio", {
  ibkr_data <- create_test_portfolio(type = "ibkr")
  result <- process_portfolio(ibkr_data)
  expect_true("IV" %in% colnames(result))
})

# Test with Gonet data (limited columns)
test_that("Works with Gonet portfolio", {
  gonet_data <- create_test_portfolio(type = "gonet")
  result <- process_portfolio(gonet_data)
  # Should handle missing columns gracefully
  expect_false("IV" %in% colnames(gonet_data))
})
```

## Debugging Test Failures

### Common Issues

1. **Module namespace errors**:
   - Verify R_BOX_PATH: `Sys.getenv("R_BOX_PATH")`
   - Check module path matches file location

2. **Reactive validation errors**:
   - Wrap optional module calls in `tryCatch()`
   - Add `req()` guards in reactives

3. **Data type mismatches**:
   - Check for logical NA vs numeric NA (`NA_real_`)
   - Verify column types with `str(data)`

4. **Missing dependencies**:
   - Run `renv::status()` to check environment
   - Install missing packages: `renv::install("<package>")`

### Verbose Output

```r
# Run tests with detailed output
testthat::test_dir(".", reporter = "progress")

# Or with stop-on-failure
testthat::test_dir(".", reporter = "stop")
```

## Test-Driven Development Workflow

1. **Write test first** (define expected behavior)
2. **Run test** (should fail initially)
3. **Implement feature** (minimal code to pass test)
4. **Run test again** (verify it passes)
5. **Refactor** (improve code while keeping tests green)
6. **Repeat** for next feature

## Integration with Git Workflow

After tests pass:

```bash
cd <AppDir> && git status
cd <AppDir> && git add .
cd <AppDir> && git commit -m "Add feature X with tests"
cd <AppDir> && git push
```

## Performance Testing

For performance-critical code:

```r
# Use microbenchmark for timing
library(microbenchmark)

microbenchmark(
  compute_greeks(portfolio),
  times = 100
)

# Or profvis for profiling
library(profvis)

profvis({
  # Run slow operation
  result <- process_large_portfolio(data)
})
```

## See Also

- `TEST.md` - Comprehensive testing guidance
- Application test files: `RReporting/test_automated.R`, `RPreTrade/test_automated.R`, etc.
- Module test examples: `Tuser/analysis/test/`
