---
"@chatcops/widget": patch
---

`autoOpen` and `welcomeBubble.delay` passed to `init()` no longer fire at once when the number is out of range. A delay past what `setTimeout` holds (2,147,483,647 ms, about 24.8 days) waits that long, `autoOpen` with `NaN`, a negative number or a non-number doesn't auto-open, and the bubble delay falls back to its default of 3000 ms. The script tag attributes were already checked.
