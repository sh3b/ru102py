
Challenge #7

As far as I'm concerned, this is the most interesting challenge of the course. I hope you'll give it a try!

Your challenge is to create a sliding window subclass of RateLimiterDaoBase. You can call it SlidingWindowRateLimiter. You'll find a bare bones implementation of the hit method in redisolar/dao/redis/sliding_window_rate_limiter.py:

    def hit(self, name: str):
        """Record a hit using the rate-limiter."""
        # START Challenge #7
        # END Challenge #7

A sliding window needs to record a timestamp for each request. You could make this work with either a stream or a sorted set, but to start, you should use a sorted set.

You'll use three sorted set commands:

    ZADD
    ZREMRANGEBYSCORE
    ZCARD

Here's the algorithm:

1. Add an entry to the sorted set with the current timestamp in milliseconds as its score. You can put whatever data you like for the element value, but the element should be unique. You could write something like [timestamp]-[ip-address] or [timestamp]-[random-number].
2. Remove all entries from the sorted set that are older than CURRENT_TIMESTAMP - WINDOW_SIZE (where WINDOW_SIZE is the number of seconds in your window). You can use the ZREMRANGEBYSCORE command to do this. This is how we slide the window forward in time. In other words, running this command ensures that the only elements in the sorted set will be those that fall within the sliding window.
3. Now run the ZCARD command on the sorted set. This returns the number of elements in the set, which is now equal to the number of requests in the sliding window.

You should perform the above three operations inside a transaction. You can then compare the return value of ZCARD to determine whether to raise RateLimitExceededException.
Key Naming

Recall that all keys for this application live in the file redisolar/dao/redis/key_schema.py. We've provided a method named sliding_window_rate_limiter_key to generate the keys for your sliding window implementation in that file.

Take a look at the method and study the structure of the key.

Notice that it has the form [limiter]:[name]:[window_size_ms]:[max_hits]. The key for the sliding window rate limiter doesn't need a minute block, like the fixed rate-limiter key. Instead, this key includes the window size in milliseconds in the name, to make it unique.
Tests

We've included tests to help you check your work. To see them, open up tests/dao/redis/test_sliding_window_rate_limiter.py and scroll down to the comment beginning with "Challenge #7". Remove the @pytest.mark.skip decorations in the three test cases that follow to allow pytest to run them.

You can run just this test suite from the command line as follows:

    $ pytest tests/dao/redis/test_sliding_window_rate_limiter.py

Don't forget that if you have any questions, you can reach us on Discord.
