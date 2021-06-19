Challenge #3 Review

In Challenge #3, we asked you to write an optimized and correct update() method for the class SiteStatsDaoRedis. Here is one solution to that problem:

# Challenge #3

    def _update_optimized(self, key: str, meter_reading: MeterReading,
                          pipeline: redis.client.Pipeline = None) -> None:
        execute = False
        if pipeline is None:
            pipeline = self.redis.pipeline()
            execute = True
    
        reporting_time = datetime.datetime.utcnow().isoformat()
    
        pipeline.hset(key, SiteStats.LAST_REPORTING_TIME, reporting_time)
        pipeline.hincrby(key, SiteStats.COUNT, 1)
        pipeline.expire(key, WEEK_SECONDS)
    
        self.compare_and_update_script.update_if_greater(
          pipeline, key, SiteStats.MAX_WH, meter_reading.wh_generated)
        self.compare_and_update_script.update_if_less(
          pipeline, key, SiteStats.MIN_WH, meter_reading.wh_generated)
        self.compare_and_update_script.update_if_greater(
          pipeline, key, SiteStats.MAX_CAPACITY, meter_reading.current_capacity)
    
        if execute:
            pipeline.execute()

Let’s walk through this solution.

First, as a review, while working on your solution you would have changed the update() method of SiteStatsDaoRedis to call the _update_optimized() method instead of _update_basic().

You can also see in the update() method that the key we’ll use is specific to the site ID and timestamp of the meter reading:

    key = self.key_schema.site_stats_key(meter_reading.site_id,
                                         meter_reading.timestamp)

Going back to the _update_optimized() method, you can see that it starts and ends with the pipeline-management code that we provided for you. The method uses the pipeline that the caller passed in, or else creates a pipeline and uses that.

But let’s get back to the important part of what this method does. Look at the first steps _update_optimized() takes with its pipeline:

    reporting_time = datetime.datetime.utcnow().isoformat()
    
    pipeline.hset(key, SiteStats.LAST_REPORTING_TIME, reporting_time)
    pipeline.hincrby(key, SiteStats.COUNT, 1)
    pipeline.expire(key, WEEK_SECONDS)

Here, we get the current time in ISO 8601 format and set that in the site stats hash identified by the key we're using. Then we increment the meter_reading_count field in the site stats hash.

Finally, we expire the entire site stats hash after one week, to avoid keeping this data around longer than we need it.

Next comes the biggest change from _update_basic() -- a series of operations we’re going to run by calling a Lua script. Let’s look closer at one of these operations. __All of the Lua script calls work the same way, so we only have to review one to learn the technique.__

    self.compare_and_update_script.update_if_greater(
      pipeline, key, SiteStats.MAX_WH, meter_reading.wh_generated)

First, where is self.compare_and_update_script defined? It’s defined in the __init__() method of the class we're looking at -- SiteStatsDaoRedis:

    self.compare_and_update_script = CompareAndUpdateScript(self.redis)

What's going on here? We're creating an instance of a Python class that represents the Lua script we want to call -- CompareAndUpdateScript -- and setting it as an instance variable. Recall that the action happened in the method update_if_greater() from that class, so let's check it out in the file redisolar/scripts/compare_and_update.py.

The CompareAndUpdateScript class is mostly a wrapper around redis-py’s Script class. Script manages registering a Lua script with Redis and calling the SHA that Redis assigned to the script. We'll see how that works by stepping through the remaining method calls, starting with update_if_greater():

    def update_if_greater(self, pipeline: Pipeline, key: str, field: str,
                          value: float) -> None:
        self.update(pipeline, key, field, value, ScriptOperation.GREATER_THAN)

Well, this is a bit of a Matryoshka doll, isn't it? So this method calls the self.update() method, which looks like this:

    def update(self, pipeline: Pipeline, key: str, field: str, value: float,
               op: ScriptOperation) -> None:
        self.script(keys=[key], args=[field, str(value), op.value],
                    client=pipeline)

Now we're getting somewhere. Here, self.script is an instance of the Script class that redis-py provides.

Because we passed self.script a pipeline object when we called it, the script will execute as part of our pipeline.

Finally, how about the Lua script itself -- what does that look like? You can read the whole script in the file redisolar/scripts/compare_and_update.lua. But there are two important parts for the update_if_greater() method that we're looking at.

First is a call to HGET to get the current value of the field we may update:

    local current = redis.call('hget', key, field)

So you can see, if the new value is greater than the currently-set value, this Lua script ends up running two operations: HGET and HSET.

And because we used the pipeline parameter when we called the Script object (which is callable like a function), redis-py queued up the call to the Lua script on our pipeline.

That means that when we finally execute the pipeline, all of the queued Lua script executions will run, and any HGET and HSET commands that the script calls execute will run within the context of our pipeline.

That's going to save considerable latency time because otherwise, we would have had to make several calls to Redis to achieve the same result.
