In multiple places, you've probably seen the warning that there aren't safeguards on your usage of the server's systems. While sometimes, preventing this is as trivial as capping a "count" parameter or making sure a rank range isnt too large, it can be a bit more difficult to make sure people aren't sending *too many* requests. Another form of strain on your server can just come from there being large amounts of requests, so to help you combat this, we've given you a few forms of limiters that you can use to prevent users from making too many requests. For example, say you had a button available to users that would load more of the top players on the leaderboard. Whenever a user presses this, you have to make a request. That can quickly add up. This should help.

We provide two types of limiters: **Rate Limiters** and **Bucket Limiters**

- **Rate Limiters** force a fixed delay between requests. For example, a rate limiter that limits the rate to 2 times per second would make sure that there is a 0.5 second delay between requests. This is better for requests that are more spread out.
- **Bucket Limiters** serve as a resource. Requests empty a bit of the bucket, and the bucket slowly fills on its own. Requests are not allowed when it would empty the bucket. This would, for example, allow a burst of requests followed by a gap in requests where the bucket "refills". This is better for requests where many of them will happen in a short period of time on occasion.

You can find the module for both types of limiter in the RankSwift folder. Just require one of the modules and run the `.new` function to create a limiter. Both limiters allow users to be specified. This allows you to have players hit limits independently. In addition, you can specified a shared limit that applies to everyone. If you just want to share a limit between all players, make the user field some constant value such as `true`. The user field can take on any type other than `nil`.

## Rate Limiter
### Events
#### used
`used`
> Fires when the rate limiter is used. Passes the user that used it.

### Methods

#### new
```lua
function Limiter.new(rate: number, shared_rate: number?): RateLimiter
```

> `rate` and `shared_rate` are in uses/second. If `shared_rate` is `nil` then a shared rate is not imposed.

#### CanUse
```lua
function limiter:CanUse(user): boolean
```

> Returns whether or not the specified user can make a request at this time.

#### Use
```lua
function limiter:Use(user)
```

> Puts the specified user on cooldown. This does not check `CanUse`, so it is suggested to check that before using `Use`.

#### ResetAll
```lua
function limiter:ResetAll()
```

> Resets the cooldowns of all users, as well as the shared cooldown.

#### Reset
```lua
function limiter:Reset(user)
```

> Resets the cooldown of the specified user, but not the shared cooldown.

#### GetRate
```lua
function limiter:GetRate(): number
```

#### SetRate
```lua
function limiter:SetRate(rate: number)
```

#### GetSharedRate
```lua
function limiter:GetSharedRate(): number?
```

#### SetSharedRate
```lua
function limiter:SetSharedRate(rate: number)
```

#### RemoveSharedRate
```lua
function limiter:RemoveSharedRate()
```

> Disables the shared rate limit functionality, rate limit is now entirely individual.

#### Destroy
```lua
function limiter:Destroy()
```

> Cleans up the limiter and its attached event.

## Bucket Limiter
### Events
#### used
`used`
> Fires when the rate limiter is used. Passes the user that used it as well as the amount used.

### Methods

#### new
```lua
function Limiter.new(bucket_size: number, regen_rate: number, shared_bucket_size: number?, shared_regen_rate: number?): BucketLimiter
```

> `regen_rate` and `shared_regen_rate` are in bucket_unit/second. If `shared_bucket_size` is `nil` then the shared limit is not imposed. If `shared_regen_rate` is `nil`, the shared bucket will not regenerate.

#### CanUse
```lua
function limiter:CanUse(user, amount: number?): boolean
```

> Returns whether or not the specified user can consume `amount` of bucket resource at this time. `amount` defaults to `1`.

#### Use
```lua
function limiter:Use(user, amount: number?)
```

> Consumes `amount` usage from both the user's bucket and the shared bucket. `amount` defaults to `1`. This does not check `CanUse`, so it is suggested to check that before using `Use`.

#### GetConsumption
```lua
function limiter:GetConsumption(user): number
```

> Returns the amount of the bucket that the specified user has consumed.

#### GetSharedConsumption
```lua
function limiter:GetSharedConsumption(user): number
```

> Returns the amount of the shared bucket that has consumed.


#### ResetAll
```lua
function limiter:ResetAll()
```

> Resets the buckets of all users, as well as the shared bucket.

#### Reset
```lua
function limiter:Reset(user)
```

> Resets the bucket of the specified user, but not the shared bucket.

#### GetBucketSize
```lua
function limiter:GetBucketSize(): number
```

#### SetBucketSize
```lua
function limiter:SetBucketSize(size: number)
```

#### GetSharedBucketSize
```lua
function limiter:GetSharedBucketSize(): number
```

#### SetSharedBucketSize
```lua
function limiter:SetSharedBucketSize(size: number)
```

#### RemoveSharedBucketSize
```lua
function limiter:RemoveSharedBucketSize()
```

> Disables the shared bucket functionality, this limiter is now entirely individual.

#### GetRegenRate
```lua
function limiter:GetRegenRate(): number
```

#### SetRegenRate
```lua
function limiter:SetRegenRate(size: number)
```

#### GetSharedRegenRate
```lua
function limiter:GetSharedRegenRate(): number
```

#### SetSharedRegenRate
```lua
function limiter:SetSharedRegenRate(size: number)
```

#### RemoveSharedRegenRate
```lua
function limiter:RemoveSharedRegenRate()
```

> Disables the regeneration of the shared bucket.

#### Destroy
```lua
function limiter:Destroy()
```

> Cleans up the limiter and its attached event.