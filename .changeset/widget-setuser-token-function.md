---
"@chatcops/widget": patch
---

`ChatCops.setUser({ token })` takes a function (sync or async) that returns a fresh token, the same as `userToken`. Passing the same function again keeps the chat; a different function, a string or no token starts a new one.
