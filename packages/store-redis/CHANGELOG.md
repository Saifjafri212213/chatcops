# @chatcops/store-redis

## 1.0.0

### Minor Changes

- 5bd99f4: Add Redis storage adapter (`@chatcops/store-redis`) supporting `@upstash/redis` and `ioredis`.

  - Implements `conversations` with atomic version-checked appends, in-place message updates, removals, listing, and purging using Redis hashes, lists, sorted sets, and Lua scripts.
  - Implements `counters` with atomic increments and expiry initialization for rate limits.
  - Implements `kv` with JSON serialization and `setIfAbsent` (`NX`) for distributed locks.
  - Implements `events` with hourly count hashes for analytics event recording and time-bucketed queries.
  - Exposes `redisStorage`, `fromIoRedis`, and the minimal `RedisLike` interface.

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
