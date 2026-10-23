# @chatcops/store-cloudflare

## 1.0.0

### Minor Changes

- 63680cf: `extraColumns` and `rowHook` stamp columns you've added (a tenant or site id) on every new conversation and message row, as in the Postgres and SQLite stores. The `kv` option is gone: ChatCops only uses key-value storage for turn locks, which need the atomic `setIfAbsent` that Workers KV doesn't have, so turn locks always live in D1 and hold across isolates.
- 131099b: Add Cloudflare storage for D1, KV, and Durable Objects counters.

### Patch Changes

- Updated dependencies [79fea7e]
- Updated dependencies [955809f]
- Updated dependencies [b138ded]
- Updated dependencies [e1e5e79]
- Updated dependencies [c2963af]
- Updated dependencies [0d666f2]
- Updated dependencies [c34a8d4]
- Updated dependencies [d320b44]
- Updated dependencies [1ca06e1]
- Updated dependencies [47ed5f1]
- Updated dependencies [aa3bb33]
- Updated dependencies [e8795ca]
- Updated dependencies [fd80586]
- Updated dependencies [049c52c]
- Updated dependencies [0578d5c]
- Updated dependencies [8e3e79e]
- Updated dependencies [1c76cf8]
- Updated dependencies [639a50c]
- Updated dependencies [6d80bc9]
- Updated dependencies [238eaef]
- Updated dependencies [332e588]
- Updated dependencies [be2ad74]
- Updated dependencies [ad1d9db]
- Updated dependencies [96372c8]
- Updated dependencies [f930e9c]
- Updated dependencies [3dd7dc8]
- Updated dependencies [545963f]
  - @chatcops/core@0.5.0
