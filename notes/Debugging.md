
Debugging

If your Python application behaves strangely in a way you suspect is related to Redis, you’ll need to start debugging. So how do you debug an application that uses Redis?
1. Check for Expired Keys

Key expiry is an important feature of Redis, but it means that your code could end up referencing a key that’s expired. So, when debugging an issue, be certain that the keys your code depends on still exist.

    Check to see which keys you're calling EXPIRE on, and be sure that the TTLs are adequate
    Check your Redis memory settings. If the maxmemory-policy setting is set to anything other than noeviction, then Redis will start evicting keys once it hits max memory

2. Try it in the CLI

The Redis CLI is perfect for testing your data access patterns. If your code isn't working as expected, try running the equivalent Redis operations using the CLI. If it doesn't work in the CLI, it's not going to work in your application, either.
3. Check RedisInsight

If you aren’t a command line person, or just want to see a visualization of your Redis database, check out RedisInsight, the official Redis Labs monitoring dashboard for all flavors of Redis.
4. Monitor the Commands Sent to Redis

If it works in the CLI but not in your application, then it's possible that the application, somehow, isn't sending the right commands or arguments to the Redis server. For that, Redis provides a command that emits a stream of all operations being sent to the server. The command is called MONITOR. You'll see how to use it in the upcoming hands-on activity.
5. Lua

Lua scripts can be tricky to debug. Redis includes a proper, fully-featured debugger for Lua that you can read about on the redis.io documentation site.
