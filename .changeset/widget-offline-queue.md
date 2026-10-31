---
"@chatcops/widget": minor
---

A message sent while the browser is offline is shown as waiting ("Will send when you're back online", the new `messageWaiting` string) instead of failing. The widget sends the waiting messages in order, one at a time, when the `online` event fires. They are never saved, and a reload or a new conversation drops them.
