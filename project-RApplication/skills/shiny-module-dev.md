# Shiny Module Development Skill

Build backward-compatible Shiny box modules following RTrading project standards.

## When to Use This Skill

- Creating new box modules in Tuser/
- Extending existing box modules with new features
- Refactoring monolithic server code into modular components

## Module Structure

All box modules must be stored under `Tuser/` (the single box repository in the system).

### Standard Module Pattern

```r
# File: Tuser/<category>/<module_name>.R
#
# @export
ui <- function(id) {
  ns <- NS(id)
  # UI elements using ns() for all IDs
  tagList(
    # ... UI code
  )
}

#' @export
server <- function(id, param1, param2, ...) {
  moduleServer(id, function(input, output, session) {
    # Server logic

    # Return list of reactive values/functions
    list(
      data = reactive({ ... }),
      action = function() { ... }
    )
  })
}
```

## Backward Compatibility Rules (CRITICAL)

When extending existing modules, backward compatibility is **MANDATORY**.

### Pattern: Optional Parameters with Defaults

```r
#' Extended module server function
#'
#' @param id Module namespace
#' @param existing_param Existing parameter (UNCHANGED)
#' @param ... Other existing parameters (UNCHANGED)
#' @param enable_new_feature Enable new feature (NEW, default = FALSE)
#' @param new_param1 New parameter (NEW, optional)
#' @param new_param2 New parameter (NEW, optional)
#'
#' @export
server <- function(id, existing_param, ...,
                   enable_new_feature = FALSE,
                   new_param1 = NULL,
                   new_param2 = NULL) {
  moduleServer(id, function(input, output, session) {

    # EXISTING FUNCTIONALITY - UNCHANGED
    # ... keep all existing code working exactly as before ...

    # NEW FUNCTIONALITY - ONLY IF ENABLED
    if (enable_new_feature) {
      # New features here
      # ...
    }
  })
}
```

### Key Principles

1. ✅ **Add new parameters at the end** with default values that preserve old behavior
2. ✅ **Never change existing parameter names, order, or types**
3. ✅ **Use feature flags** (e.g., `enable_projection = FALSE`) to opt-in to new features
4. ✅ **Keep existing outputs unchanged**, add new outputs conditionally
5. ✅ **Extend return lists**, never replace existing items
6. ✅ **Test with all known consumers** before deploying

### Example - Existing Consumer (NO CHANGES NEEDED)

```r
# Works exactly as before
myModule$server("id", existing_param)
```

### Example - New Consumer (OPTS IN to new features)

```r
# Explicitly enables new functionality
myModule$server(
  "id",
  existing_param,
  enable_new_feature = TRUE,
  new_param1 = reactive(input$value1),
  new_param2 = reactive(input$value2)
)
```

## Module Dependencies

- Box Modules have complex interdependencies - check existing usage patterns before modifying
- Applications mix multiple box modules (e.g., `journal` + `analysis`)
- **RStudies applications** use box modules from `Tuser/studies/view/`
- Test that changes don't break downstream consumers:
  - an_lineUI
  - ROrder
  - RJournal
  - RStudies apps
  - RPreTrade
  - RReporting

## Testing Your Module

Create a standalone test script to verify the module works in isolation:

```r
# test_module.R
library(shiny)

box::use(
  category/module_name[ui, server]
)

ui <- fluidPage(
  module_name$ui("test")
)

server <- function(input, output, session) {
  module_name$server("test", param1 = reactive(...))
}

shinyApp(ui, server)
```

## Namespace Composition Strategy

When composing modules, **keep critical shared modules at top level** to avoid namespace issues:

```r
# CORRECT - an_lineUI at top level
box::use(
  analysis/view/an_lineUI
)

# Server function
an_lineUI$server("id", ...)
```

Avoid deeply nested namespace composition that can break existing references.

## Documentation

- Use roxygen2-style comments for exported functions
- Document all parameters, especially new optional ones
- Include examples showing both old and new usage patterns
- Note which applications consume this module

## Versioning

- Backward-compatible feature additions → Minor version bump (x.Y.0)
- See `RPreTrade/REFACTORING_PLAN.md` Section "Backward Compatibility Strategy" for detailed patterns

## Common Pitfalls

- **Shiny module validation errors with optional modules**:
  - Wrap optional module method calls in `tryCatch()` to catch validation errors
  - Example: When Position 2 module may not be initialized

  ```r
  pos2_data <- tryCatch(
    position2_result$lignes_completed(),
    error = function(e) {
      if (inherits(e, "validation")) return(NULL)
      stop(e)
    }
  )
  ```

- **Module namespace issues**: Always use proper `box::use()` syntax
- **R_BOX_PATH not set**: Verify `Sys.getenv("R_BOX_PATH")` points to Tuser directory
