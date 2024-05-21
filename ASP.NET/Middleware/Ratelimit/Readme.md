- Built-in rate limit middleware from .net 7
- Control amount of traffic, can improve performance of app. All users haave a fair chance to access the app.
# [Rate limit algorithms](https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit?view=aspnetcore-8.0#rate-limiter-algorithms)
- Fixed window: uses a fixed time window to limit requests. When the time window expires, a new time window starts and the request limit is reset.
- Silding window: is similar to the fixed window limiter but adds segments per window, requests added back to the current segments from the expired segment
- Token bucket: The token bucket limiter is similar to the sliding window limiter, a fixed number of tokens are added each replenishment period instead.
- Concurency: The concurrency limiter limits the number of concurrent requests
# Two kinds of rate limit
- Global limiter
- Rate limit with policies
# How to config rate limit
# Write your own rate limit policy
# Example
