---
name: Economic events calendar URL
description: Working URL to fetch weekly economic events for macro_context report section 08
type: reference
---

For updating `events.R` (section 08 of macro_context report), fetch events from Equals Money:

- **March**: `https://equalsmoney.com/economic-calendar/march`
- **April**: `https://equalsmoney.com/economic-calendar/april`
- Pattern: `https://equalsmoney.com/economic-calendar/<month>`

These URLs work with WebFetch (unlike ForexFactory which returns 403).

Events file: `RStudies/reports/macro_context/events.R` — edit every Monday.
