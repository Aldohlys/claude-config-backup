---
name: feedback-ggplotly-autosize
description: ggplotly() output renders at fixed default width inside plotlyOutput() even though container is 100% wide — must pipe through layout(autosize=TRUE) and config(responsive=TRUE)
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 20b7af92-341a-4d0c-8f35-f1b2b041303a
---

`plotly::ggplotly(plot)` produces a plotly object that does NOT inherit the container's width. The inner canvas renders at plotly's default fixed render size (typically ~half the available panel) even though `shiny::plotlyOutput()` defaults to `width = "100%"`.

**Symptom**: Shiny chart that should fill a `column(10, ...)` panel only spans ~55% of the available width; legend pinned to the right of the narrow canvas; large empty space to the right of the legend.

**Fix**:
```r
plotly::ggplotly(plot) |>
  plotly::layout(autosize = TRUE) |>
  plotly::config(responsive = TRUE)
```

**Why:** plotly.js needs `autosize = TRUE` to defer to the container, and `responsive = TRUE` to re-layout on window resize. ggplotly omits both by default.

**How to apply:** Any Shiny module wrapping a ggplot with `ggplotly()` and rendering via `plotlyOutput()`. Confirmed fixed 2026-05-12 in `analysis/view/ana_optUI.R` (Multi-Date Charts tab — RPreTrade). Other Tuser files using `ggplotly()` may have the same latent issue: `analysis/logic/analytics.R`, `account/view/accountUI.R`, `symbol/view/displaysymUI.R`, `symbol/view/plotsymUI.R`, `portfolio/view/displayportfUI.R` — audit if a user reports similar narrow-chart symptoms.
