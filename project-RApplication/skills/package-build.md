# Package Build Skill

Manual build and deployment workflow for R packages (Tbasics, Tdata, Tlogger).

## When to Use This Skill

- Manual package build when `/build` command is unavailable or inappropriate
- Understanding the build process for debugging
- Custom build scenarios requiring specific steps

**Note**: For standard package builds, prefer the `/build <package>` slash command which automates this workflow.

## Build Process Overview

```
Document → Test → Build → Deploy to renv projects → Snapshot
```

## Step-by-Step Workflow

### 1. Documentation Update

Update package documentation from roxygen2 comments:

```bash
cd <package_dir> && Rscript -e "devtools::document()"
```

**What it does**:
- Generates .Rd files in `man/` from roxygen2 comments
- Updates NAMESPACE file with exports
- Creates/updates package documentation

### 2. Run Tests

```bash
cd <package_dir> && Rscript -e "devtools::test()"
```

**What to check**:
- All tests pass (no failures or errors)
- Warning messages should be investigated
- Note: Syntax validation is automatic - don't run separate checks

### 3. Build Package Tarball

```bash
cd <package_dir> && Rscript -e "devtools::build()"
```

**Output**: Creates `../<PackageName>_<version>.tar.gz`

**Version Management**:
- Edit `DESCRIPTION` file before building
- Follow semantic versioning:
  - **Patch** (x.y.Z): Bug fixes, no API changes
  - **Minor** (x.Y.0): New features, backward compatible
  - **Major** (X.0.0): Breaking changes

### 4. Deploy to renv Projects

Install the built package in each application that depends on it:

**For Tuser (has renv)**:
```bash
cd Tuser && Rscript -e "renv::install('../<PackageName>_<version>.tar.gz')"
```

**For RStudies** (if applicable):
```bash
cd RStudies && Rscript -e "renv::install('../<PackageName>_<version>.tar.gz')"
```

**For other applications** (RReporting, RJournal, ROrder, RPreTrade):
```bash
cd <AppDir> && Rscript -e "renv::install('../<PackageName>_<version>.tar.gz')"
```

### 5. Update renv.lock

After installing, update the lock file:

```bash
cd <AppDir> && Rscript -e "renv::snapshot()"
```

**What it does**:
- Records the new package version in `renv.lock`
- Ensures reproducible environment

## Build Dependency Order

When building multiple packages, follow dependency order:

```
1. Tlogger (no dependencies)
2. Tbasics (depends on Tlogger)
3. Tdata (depends on Tlogger and Tbasics)
```

If you update Tlogger, you must rebuild Tbasics and Tdata.

## Package Structure Reference

### Tbasics
- **Purpose**: General utilities and option computations (Black-Scholes, Greeks)
- **Deploy to**: Tuser, RStudies, RReporting, RJournal, ROrder, RPreTrade

### Tdata
- **Purpose**: IBKR TWS API access and database operations
- **Deploy to**: All applications (Tuser, RStudies, RReporting, RJournal, ROrder, RPreTrade)
- **Note**: Large package, de facto backend for most RTrading applications

### Tlogger
- **Purpose**: Logging infrastructure
- **Deploy to**: All applications
- **Usage**: Wrappers on top of R `logger` package

## Verification

After deployment, verify the installation:

```r
# Check package version
packageVersion("<PackageName>")

# Load package to check for errors
library(<PackageName>)

# Check renv status
renv::status()  # Should show "No issues found"
```

## Git Integration

### Commit Changes Before Building

```bash
cd <package_dir> && git status
cd <package_dir> && git diff
cd <package_dir> && git add .
cd <package_dir> && git commit -m "Descriptive commit message"
cd <package_dir> && git push
```

**Commit Message Guidelines**:
- Focus on "why" rather than "what"
- Include context about the changes
- Reference issue numbers if applicable

## Changelog Updates

### Project CHANGELOG.md (in git)

Update `<package>/CHANGELOG.md` with:
- Version number and date
- Feature additions / Bug fixes / Breaking changes
- Technical decisions and lessons learned

Format:
```markdown
## [x.y.z] - 2025-MM-DD

### Added
- New feature description

### Fixed
- Bug fix description

### Changed
- API change description
```

### System change.log (not in git)

**Note**: `/build` command handles this automatically. For manual builds, add summary to `RApplication/change.log`:

```markdown
## 2025-MM-DD - <PackageName> v<version>: Brief summary

**Files Modified**:
- <file1>, <file2>, ...

**Testing**: X/X tests passing
```

## Troubleshooting

### Tests Fail After Dependency Update

```r
# Rebuild all dependent packages in order
cd Tlogger && devtools::build()
cd Tbasics && renv::install("../Tlogger_x.x.x.tar.gz") && devtools::build()
cd Tdata && renv::install("../Tbasics_x.x.x.tar.gz") && devtools::build()
```

### renv::status() Shows Issues

```r
# Restore from lock file
renv::restore()

# Or install missing package
renv::install("<package>")
renv::snapshot()
```

### Build Fails with "Error in loadNamespace"

- Check DESCRIPTION dependencies are correct
- Verify all imported packages are installed
- Check roxygen2 @import/@importFrom directives

## Performance Notes

- Documentation update is fast (<10 seconds)
- Testing time varies by package (Tdata tests may take 1-2 minutes)
- Building is fast (<30 seconds)
- Deployment to each renv project takes 10-30 seconds

## See Also

- `build.md` - Complete build and deployment workflow documentation
- `/build` slash command - Automated build workflow (preferred method)
- `CHANGELOG.md` in each package directory - Version history
