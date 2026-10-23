# @chatcops/core

## 0.5.0

### Minor Changes

- 79fea7e: New `chatcopsAiSdkHandler(config)`: a Fetch API handler that the AI SDK's `useChat` can talk to. It reads `useChat` requests and answers in the AI SDK's UI message stream, so an app built on the AI SDK can use a ChatCops server without changing its chat UI. The server keeps the history and reads only the last user message, so a client can't add turns the model never wrote. Server tools show up as tool parts the client doesn't run, a captured lead as a `data-lead-captured` part, and each assistant message has `metadata.conversationId`. A contract test runs the real `ai` 7 client against it.

  `@chatcops/core` exports the mapping on its own: `aiSdkEncoder()` and `fromAiSdkRequest()`. `protocolStats()` on the handler now also counts `'ai-sdk'` requests.

- 955809f: Budgets: limits on tokens, replies and dollars per conversation, per IP per UTC day, and overall per day, counted on a `CounterStore` so they hold across instances. Core has `createBudgetGuard()`, and `runTurn()` takes a `budget` that is checked before each model request, charged after it, and caps the reply length to the tokens left. A used-up budget ends the reply with `budget_exceeded` (`provider_error` for widgets before 0.6) without calling the model.
- b138ded: Add `ClaudeProvider.step()`, which runs one model request and reports text, reasoning, tool calls, token usage and why the model stopped, without running tools. Stored messages with parts go back to Claude as native blocks, including thinking signatures. Also adds a `fetch` option to every provider config (for proxies, or replaying recorded responses in tests) and `normalizeTurns()`, which cleans up history like `normalizeHistory()` but keeps tool calls and their results. `estimateTokens()` now counts tool inputs and results on messages with parts.
- e1e5e79: Add an openai-compatible provider with optional API keys, custom headers, configurable tool support and max_tokens by default. Allow applications to register custom provider factories while validating keys for hosted providers.
- c2963af: Add `createRateLimiter` — a sliding-window rate limiter built on `CounterStore`.

  - Uses two fixed-window counters to produce a smooth sliding estimate without per-request timestamps: `estimate = prev × (1 − elapsed/windowMs) + current`.
  - Counts denied requests, so a hammering client stays limited.
  - Returns `remaining` (never below 0) and `retryAfterMs` (clamped to `[1, windowMs]`) on every call.
  - No timers or in-process state — safe at global scope on Cloudflare Workers and Vercel Edge.
  - Exported from `@chatcops/core` as `createRateLimiter`, `RateLimitRule`, `RateLimitResult`, `RateLimiterOptions`.

- 0d666f2: The server has budgets on by default: 200 replies per conversation and 2M tokens per IP per UTC day (`DEFAULT_BUDGETS`, filled in by `withDefaultBudgets()`). There's no default daily total. The server logs the budgets in force once at startup. Set a scope in `budgets` to replace its default, set a scope to `false` to turn it off, or pass `budgets: false` to turn them all off.
- d320b44: Map protocol error codes to HTTP statuses (`statusForError`), build wire error bodies (`errorBody`), and format validation error paths (`describeInvalidRequest`).
- 1ca06e1: Add `GeminiProvider.step()`. Thought signatures stay on the parts they came with and go back to Gemini on the next step. Calls from another provider get the placeholder signature Google documents, so a conversation can switch to Gemini 3 partway through. Safety, recitation and blocklist stops are reported as `content-filter`. Function call ids that Gemini didn't send are made up locally and never sent back.
- 47ed5f1: Tools can describe their input with a full JSON Schema in `inputSchema`: nested objects, arrays, `enum`, `anyOf` and `$ref` all go to Claude, OpenAI and Gemini as written. The flat `parameters` form from 0.4 still works and is converted to the same schema.

  - `execute(input, context)` now gets a second argument with the `conversationId` and an `AbortSignal` that fires when the visitor leaves. Tools that ignore it keep working.
  - Tools can carry `annotations` (`title`, `readOnlyHint`, `destructiveHint`).
  - The server checks tool definitions when it starts and throws a `ToolDefinitionError` that lists every problem: an invalid or duplicate name, an `inputSchema` whose root isn't `type: 'object'`, both `inputSchema` and `parameters` set, or a `required` key that isn't in `parameters`. Before, these failed as a provider error on the first message.
  - New exports: `toolInputSchema`, `legacyParametersToSchema`, `validateTools`, `ToolDefinitionError`, and the `ToolContext` and `ToolAnnotations` types. `ToolDefinition.parameters` and `required` are now optional.

- aa3bb33: Importing `@chatcops/core` no longer loads the Claude, OpenAI and Gemini SDKs, which cuts serverless cold-start time. To get there, `ClaudeProvider`, `OpenAIProvider` and `GeminiProvider` are no longer exported from the package root. Import them from their own subpaths instead: `@chatcops/core/providers/claude`, `@chatcops/core/providers/openai` and `@chatcops/core/providers/gemini`. `createProvider()` is unchanged and still loads only the SDK for the provider you configure.
- e8795ca: Add `toLegacyChunk(part)`, which turns a protocol v1 stream part into the chunk a widget before 0.6 reads (`{"content":"Hi"}`, `{"done":true}`, `{"error":"rate_limit","retryAfter":2}` and so on), or `null` for parts those widgets have no use for. The 0.6 server will use it to keep answering old widgets.
- fd80586: Conversations are written with `append` instead of rewriting the whole conversation, so two server instances answering the same conversation no longer lose each other's messages, when the store supports it.

  - `ConversationManager` takes `storage` (a `ChatCopsStorage`) as well as `store`, and defaults to `memoryStorage()`, which is bounded, instead of the unbounded `MemoryStore`.
  - A store with only `get`, `save` and `delete` still works; the manager adds the other methods with `withAppend()` and logs one warning that concurrent writes from two instances can lose a message.
  - A conversation is saved with its first message, not when it's first read.
  - The history sent to the model is trimmed by an estimated token budget (32,000 by default, `maxHistoryTokens`), keeping the visitor's first message and the newest ones, instead of by message count. `trimHistory()` and `estimateTokens()` are exported. The stored cap (`maxMessages`, 100) still applies, now through atomic removes, and still never leaves an assistant turn first.
  - New `appendMessages()` (with `expectedVersion`), `updateMessage()`, `getHistory()` and `acquireTurn()`, which keeps two replies from being generated for one conversation at once, across instances when the storage has `kv`.

- 049c52c: Add the types the next releases build on, with no change in behavior yet:

  - `ChatMessage` can carry `parts` (text, reasoning, sources, tool calls and results, captured leads, handoff), a `status` (`complete`, `incomplete`, `aborted`), `usage` and `aiGenerated`. `Conversation` can carry a `version`. All of them are optional, so stores written for 0.4 keep working. `partsOf(message)` reads the parts of any message, treating `content` as one text part for messages saved without them, and `messageText(parts)` joins the text parts.
  - `ProviderEvent`, `ProviderStepParams` and an optional `step()` on `AIProvider`, for providers that run one model request at a time.
  - Wire protocol v1 types (`WirePart`, `ChatRequestV1`, `WireErrorCode`, `WireErrorBody`) and `PROTOCOL_VERSION`.
  - The built-in providers now expose their `model` as a read-only property.

- 0578d5c: Add `OpenAIProvider.step()`. It asks for token usage with `stream_options.include_usage`, and if an OpenAI-compatible server rejects that option, it retries once without it and stops asking. A step that ends with `stop` but has tool calls (Ollama does this) counts as a tool step. Arguments that aren't valid JSON are reported with an `inputError` instead of being run.
- 1c76cf8: Claude requests now mark the system prompt and tools as a cached prefix, so repeat requests read them from the prompt cache. Turn it off with `promptCaching: false`. Steps and `runTurn()` take a new `systemContext` for instructions that change per request (knowledge base excerpts, the visitor's language), sent after the cached prefix so it stays the same from one request to the next. Cache reads and writes are reported in `usage`.
- 639a50c: The server speaks protocol v1. A request with `"protocol": 1` in the body (what the 0.6 widget sends) or an `x-chatcops-protocol: 1` header gets typed parts (`start`, `text-delta`, `finish` and so on) and a response header `x-chatcops-protocol: 1`. Requests without it get exactly the 0.5 stream, and the server logs one line the first time it answers one.

  In protocol v1, a request that can't be answered gets a real HTTP status with a JSON body and CORS headers: 400 `invalid_request`, 409 `conversation_busy`, 413 `payload_too_large` (bodies over 256 KB), 429 `rate_limited` with `Retry-After`, 503 `provider_unavailable`. Errors after the stream starts are an `error` part, then `finish`.

  All three adapters now share one module for this, so they send the same headers (`Content-Type: text/event-stream; charset=utf-8`, `Cache-Control: no-cache, no-transform`, `X-Accel-Buffering: no`, and no `Connection`, which HTTP/2 doesn't allow) and one `[DONE]`. When nothing has been sent for 10 seconds, as during a slow tool, they send a `: ping` comment so proxies and the widget don't close the connection. The CORS preflight allows the `Authorization` and `X-ChatCops-Protocol` headers.

  New: `exposeUsage: true` adds token usage to the v1 `finish` part. `createChatHandler()` has `startChat()`, which resolves a refusal or the reply's v1 parts, for custom adapters, and `protocolStats()`, which counts requests in each format. `@chatcops/core/protocol` exports the protocol types and helpers on their own, for clients that don't need the rest of core.

- 6d80bc9: The server's `provider` option also takes a provider object (anything with `step()` or `chat()`), for providers ChatCops doesn't ship and for tests with `scriptedProvider()`. Core exports `isAIProvider()` to tell the two apart.
- 238eaef: Add buffered analytics event writing and sink-backed statistics.
- be2ad74: Add `runTurn()`, the loop that runs one reply: model steps, the tools the model asks for, and saving the reply as it grows. It yields protocol v1 parts and returns the reply as a `ChatMessage` with parts, status and token usage. It works with providers written for 0.4 that only have `chat()`, too.

  `chat()` and `chatSync()` on the Claude, OpenAI and Gemini providers now run on `runTurn()`. They behave as before, with three differences: `chatSync()` streams under the hood, a tool timeout now aborts the tool's signal, and an OpenAI-compatible server that ends a tool step with `stop` gets its tools run.

  Also new: `toolRegistry()`, an optional `emits()` on tools (`LeadCaptureTool` uses it to add a `lead-captured` part), and `scriptedProvider()` at `@chatcops/core/testing/provider` for testing without a model.

- ad1d9db: Session tokens. With `sessionSecrets` set, `POST <chat url>/session` gives a short-lived token bound to one conversation and the page's origin, and chat requests need it as `Authorization: Bearer <token>` (401 `session_required`, `token_expired` or `conversation_mismatch` otherwise). A conversation that has messages is resumed only with a token that was issued for it, so a guessed id gets a new conversation. An id with nothing stored yet is given out as is. `requireSession: false` lets requests without a token through while older widgets are around. Adds the `conversation_mismatch` error code to protocol v1.
- 96372c8: Expose shared SSE utilities for streaming responses:

  - `encodeSSE(part)`: encodes a `WirePart` or object into standard SSE framing (`data: <json>\n\n`).
  - `SSE_PING`: comment frame (`: ping\n\n`) to keep connections alive without emitting events.
  - `SSE_DONE`: terminal frame (`data: [DONE]\n\n`) signaling stream completion.
  - `SSE_HEADERS`: immutable standard HTTP headers for SSE responses (`Content-Type`, `Cache-Control`, and `X-Accel-Buffering`, with `Connection` omitted for HTTP/2 compatibility).

- f930e9c: Add the storage layer the next server release uses. Nothing uses it yet, so behavior doesn't change.

  - `ConversationStore` gains optional `append` (atomic, with an `expectedVersion` check that throws `ConflictError`), `updateMessage`, `removeMessage`, `list` and `purge`. A store with only `get`, `save` and `delete` is still valid.
  - New `CounterStore`, `KeyValueStore` and `EventSink` interfaces, grouped in `ChatCopsStorage`.
  - `memoryStorage()` implements all of them in process, with a cap on conversations (least recently used go first), idle expiry, capped counters, keys and events, and no timers, so it's safe at global scope on Workers.
  - `withAppend(store)` adds the new conversation methods to a 0.4 store, serializing writes per conversation within one process.
  - `@chatcops/core/testing` exports `runStorageConformance(name, factory)`, the checks every store must pass, for store packages and custom stores. It needs `vitest`, an optional peer dependency.

- 545963f: Cloudflare Turnstile when a session is issued. With `turnstile: { secretKey }` on the server, `/session` checks a Turnstile token with Cloudflare before signing (401 `turnstile_failed` otherwise, 503 when Cloudflare can't be reached), and a refresh with the previous token skips it. The widget's `turnstileSiteKey` (`data-turnstile-site-key`) loads Cloudflare's script only when it first needs a session and renders the challenge in the page, outside the shadow root.

### Patch Changes

- c34a8d4: `chat()` and `chatSync()` on `ClaudeProvider`, `OpenAIProvider` and `GeminiProvider` are marked `@deprecated` in their types, pointing to `runTurn()`. They behave as before.
- 8e3e79e: Add `homepage`, `bugs`, `keywords` and `engines` (`node >=22`) to the package metadata, so npm links to the docs and the issue tracker. Core and server are marked `sideEffects: false` for tree shaking. The widget isn't, because importing it starts the widget.
- 332e588: Regenerate can name the reply to replace: `regenerate: { assistantMessageId }`. `true` still works. Retrying a message whose reply was already saved sends that reply again instead of asking the model for a second one, and a partial reply is replaced once a new one succeeds. A reply the visitor walked away from is now saved, marked `aborted`, and left out of what the model sees next.
- 3dd7dc8: Use random UUIDs for fallback tool call IDs in OpenAI and Gemini providers instead of module counters.

## 0.4.0

### Minor Changes

- f406cf9: Stop the provider when the visitor disconnects. `ProviderChatParams` takes a `signal`, which all three providers pass to their SDK and check between tool rounds. `handleChat(body, clientIp, { signal })` stops quietly on abort and doesn't save the partial reply. The Express adapter aborts when the response closes early; the Vercel and Cloudflare adapters abort when the response stream is cancelled, and the Cloudflare adapter no longer throws an unhandled rejection after a disconnect. The Cloudflare handler accepts the Workers `(request, env, ctx)` arguments and uses `ctx.waitUntil` so lead webhooks still finish.
- e694ba4: Upgrade the provider SDKs and the default models.

  - Claude: `@anthropic-ai/sdk` 0.135. The default model is now `claude-haiku-5-5`. Thinking blocks are sent back unchanged between tool rounds, and a refusal or a context window overflow is reported as an error instead of an empty reply.
  - OpenAI: `openai` 7. The default model is now `gpt-5-mini`. Requests send `max_completion_tokens` (set `maxTokensParam: 'max_tokens'` for servers that need the old name), and tool calls run in parallel like the other providers.
  - Gemini: moves from the deprecated `@google/generative-ai` to `@google/genai`. The default model is now `gemini-3.5-flash-lite`, because Google shut down the old default `gemini-2.0-flash`. Function results carry the call id, and thought signatures are kept between tool rounds.
  - New provider options: `effort` (default `'low'`), `baseURL` for proxies and OpenAI-compatible servers such as Ollama or OpenRouter, `organization` for OpenAI, and `maxTokensParam`.
  - `temperature` is left out, with a one-time warning, on models that reject it (Claude 5.x, Claude Opus 4.7 and later, OpenAI reasoning models).
  - The default `maxTokens` goes from 1024 to 4096, because thinking tokens count toward it.
  - History sent to a provider now always starts with a user message, and consecutive messages from the same side are merged. This fixes Gemini errors after about 50 exchanges and after a failed reply. `normalizeHistory` is exported.

  Migration: if you relied on the old default models, set `model` explicitly. `@google/generative-ai` is no longer installed with `@chatcops/core`; add it to your own dependencies if your code imports it directly.

- 3a4b9b8: Webhook deliveries are now signed over the raw body and can be verified with the new `verifyWebhook()` helper. Each request has an `X-ChatCops-Signature: t=<seconds>,v1=<hmac>` header, and the body is `{ id, type, created, data }` with a stable event id for deduplication (the old `event` and `timestamp` fields are still there; the `signature` field in the body is gone). 429 responses are retried and `Retry-After` is honored, redirects aren't followed, and a delivery that fails for good is logged without the body. Webhook URLs must be https and can't point at loopback, private or link-local addresses unless the webhook sets `allowPrivateNetwork: true`. The reply's `done` event is now sent before waiting for webhooks, so a slow receiver no longer delays the end of the visitor's reply.

### Patch Changes

- a1829e6: Errors that were swallowed before now show up in the server logs with the `[chatcops]` prefix: unexpected errors in the Express, Vercel and Cloudflare adapters, and a lead capture callback that throws. Logs carry the error and where it happened, never request bodies or lead data. Errors after a visitor disconnects aren't logged. What visitors see is unchanged.

## 0.3.2

### Patch Changes

- ce073a4: Fix missing accents in the Spanish, French and German locale strings (including the opening ¿/¡ in Spanish questions and exclamations) and write the Hindi locale in Devanagari instead of romanized text.

## 0.3.1

### Patch Changes

- 5ab63bc: Strip punctuation from knowledge base queries so trailing characters like "?" no longer prevent a match.

## 0.3.0

### Minor Changes

- b650792: Add retry and regenerate support for widget conversations, including assistant message regeneration, retryable error bubbles, and server-side conversation updates for regenerate requests.

### Patch Changes

- 45d02ae: Execute provider tool calls during chat loops so streaming and sync responses can continue after tool use. This also wires successful lead-capture tool executions into the server analytics, webhook flow, and widget lead-captured callbacks/events.
- 4ce9d5a: Add a configurable timeout guard for tool execution so hanging tools fail gracefully instead of blocking provider responses indefinitely.

## 0.2.1

### Patch Changes

- 959b5f2: Add README.md to all packages with install instructions, usage examples, and API reference

## 0.2.0

### Minor Changes

- ee92c50: Initial release of ChatCops packages with AI provider abstraction, server adapters, and embeddable chat widget.
