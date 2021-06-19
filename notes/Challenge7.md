Challenge #7 Review

In Challenge #7, we challenged you to create a sliding window rate limiter.

Recall the general shape of the algorithm we detailed for the challenge:

1. Add an entry to the Sorted Set with the current timestamp in milliseconds as its score.
2. Remove all entries from the Sorted Set that are older than *CURRENT_TIMESTAMP - WINDOW_SIZE*. 
3. Run the *ZCARD* command on the Sorted Set to get the number of requests in the sliding window.

Here is one solution to the challenge:

    def hit(self, name: str):
        key = self.key_schema.sliding_window_rate_limiter_key(name, self.window_size_ms,
                                                              self.max_hits)
        now = datetime.datetime.utcnow().timestamp() * 1000
        
        pipeline = self.redis.pipeline()
        member = now + random.random()
        pipeline.zadd(key, {member: now})
        pipeline.zremrangebyscore(key, 0, now - self.window_size_ms)
        pipeline.zcard(key)
        _, _, hits = pipeline.execute()
        
        if hits > self.max_hits:
            raise RateLimitExceededException()

Let's walk through the example to break down how it works.
The Sorted Set Key

    key = self.key_schema.sliding_window_rate_limiter_key(name, self.window_size_ms, self.max_hits)

Like the fixed rate-limiter, the hit() method on the sliding window rate-limiter accepts a name. To produce a key for the Sorted Set we use, we combine that name with the configured window size in milliseconds and the max number of hits per window.

In your applications, you might include more information to distinguish these keys -- like the username or token.
The Transaction

It's important that the code that adds hits to the Sorted Set, trims it, and finds the number of current hits runs inside a transaction, so that it’s atomic. You can see that the example app does with the pipeline() method.

    pipeline = self.redis.pipeline()
    member = now + random.random()
    pipeline.zadd(key, {member: now})
    pipeline.zremrangebyscore(key, 0, now - self.window_size_ms)
    pipeline.zcard(key)
    _, _, hits = pipeline.execute()

Remember that redis-py's pipeline() method runs a transaction by default, not a pipeline like you might expect!
The Sorted Set Member

Recall that Sorted Sets take a member and a score. For the member, we need a way to distinguish hits that are close together in time. Our example keeps it simple by using the current time in milliseconds, plus a random number:

    now = datetime.datetime.utcnow().timestamp() * 1000
    # ...
    member = now + random.random()

Why the random number? That's because computers are fast, especially while running tests on your laptop against a local Redis server, so you might get multiple hits within a microsecond. The random number added to the timestamp allows the rate-limiter to see these hits as distinct members.

If you're wondering how this could possibly work if we're adding random numbers to the member, the key thing to understand is that we're going to look up hits by score, not by member.

So it’s okay -- and preferable -- that multiple hits generate multiple Sorted Set members, even if the score is the same.
The Score

The crux of this rate-limiter, when you implement it as a Sorted Set, is the score.

    pipeline.zadd(key, {member: now})

With redis-py, you pass a dictionary to the zadd() method to add to a Sorted Set. And here, we've used our time-plus-random-number value as the member of the Sorted Set, and the current timestamp as the score.

Multiple hits will end up having unique members, but the same score, which will allow us to find the number of hits in the window by trimming members outside of the window (by their score) and then running ZCARD, which is exactly what we do next.
Trimming the Sorted Set

This technique works because we can trim a Sorted Set by score. Here's how we do it in the solution:

    pipeline.zremrangebyscore(key, 0, now - self.window_size_ms)

We trim everything from 0, or the beginning of time according to our Sorted Set scores, up until the start of the current window. We know the start of the window is the present moment in milliseconds, minus the number of milliseconds in our window.
Getting the Number of Hits

Now all that’s left to do is find the number of hits in the current window, which we can do with ZCARD.
    
    pipeline.zcard(key)
    _, _, hits = pipeline.execute()

The underscore (_) character is what we generally use in Python programs to say that a value is ignored. Pipelines return a value for every command run, so here we ignore everything except the final ZCARD command.

Remember that ZCARD gives us the cardinality of the Sorted Set, or in other words, the number of members. Since we've trimmed all members outside of the current window, what we get back from ZCARD is the number of hits in the current window.

Now that we have the number of hits, we can check if it’s over the threshold and return an error if so:

    if hits > self.max_hits:
        raise RateLimitExceededException()

And... that should solve it!
