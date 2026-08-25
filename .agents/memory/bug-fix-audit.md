---
name: Comprehensive bug fix audit
description: 8 bugs fixed in the nkxfca library — root causes and decisions worth remembering for future changes.
---

## Fixes applied

**sendMessage.js reduce() logic**
- `{ threadID: v.thread_fbid } || p` is always truthy — `p` fallback was never used. Fixed: gate on `v.message_id || v.thread_fbid` before returning the object.

**markAsRead.js raw object throw**
- `throw { error: "..." }` produces a non-Error with no stack trace. Fixed: `throw new Error(...)`. Any raw object thrown in a catch handler will confuse callers.

**axios.js ERROR_COOLDOWNS[368] too short**
- Error 368 = "Action Blocked" — Facebook's hardest rate-limit signal. 120s cooldown is far too short; bot re-triggers the block almost immediately. Fixed to 3600000ms (1 hour).
- **Why:** 368 is a content-send block; Facebook won't lift it in under an hour regardless of backoff.

**antiSuspension.js circuit breaker threshold**
- `maxSignalsBeforeTrip: 2` tripped the 45-minute circuit breaker on every two back-to-back errors, causing unnecessary long pauses. Fixed to 3.
- **Why:** Two signals is too sensitive for normal error variance; 3 is a meaningful pattern.

**antiSuspension.js / rateLimiter.js setInterval .unref()**
- Internal housekeeping intervals should call `.unref()` so they don't keep the Node.js event loop alive when the bot is otherwise done. Also applied to appStateBackup interval.

**rateLimiter.js MAX_REQUESTS_PER_MINUTE**
- Reduced from 80 → 50, and _MAX_PER_ENDPOINT_PER_MINUTE from 30 → 20. 80 req/min is aggressive enough to trigger bot detection.

**rateLimiter.js / monitoring.js unbounded Maps**
- `_endpointWindows` capped at 100 entries (evict oldest on overflow). `_cooldownCache` capped at 500 (evict oldest quarter on overflow). `byEndpoint` in monitoring capped at 200, `byType`/`byCode` at 100 each.
- **Why:** Long-running bots accumulate one Map entry per unique thread/endpoint forever — a slow memory leak that compounds over days.

**autoReLogin.js pendingRequests overflow**
- Added `MAX_PENDING = 200` guard. If re-login keeps failing, concurrent callers pile up in the `pendingRequests` array. Beyond 200, new callers receive an immediate rejection instead of queueing.
- **Why:** Without the guard, a stuck re-login loop with many message handlers can OOM the process.
