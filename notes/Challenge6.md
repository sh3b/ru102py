Challenge #6 Review

In Challenge #6, we asked you to complete the _insert() method from the file redisolar/dao/redis/feed.py, so that it wrote a meter reading to two separate streams: a global stream and a site-specific stream.

Here is a solution to that challenge:

    def _insert(self, meter_reading: MeterReading, 
                pipeline: redis.client.Pipeline) -> None:
        global_key = self.key_schema.global_feed_key()
        site_key = self.key_schema.feed_key(meter_reading.site_id)
        serialized_meter_reading = MeterReadingSchema().dump(meter_reading)
    
        pipeline.xadd(global_key, serialized_meter_reading, 
                      maxlen=self.GLOBAL_MAX_FEED_LENGTH)
        pipeline.xadd(site_key, serialized_meter_reading, 
                      maxlen=self.SITE_MAX_FEED_LENGTH)

Let’s walk through this step by step.

First, we get the global feed key:

    global_key = self.key_schema.global_feed_key()

Then we get the key for a site feed:

    site_key = self.key_schema.feed_key(meter_reading.site_id)

You’ve now been working with the KeySchema object and have seen how it works, so we don’t need to recap that.

Next, we need a dictionary of data for the meter reading. The way to get that in this project, as we’ve seen in other challenges, is to use a Schema class -- in this case, MeterReadingSchema.

    serialized_meter_reading = MeterReadingSchema().dump(meter_reading)

Recall that we can dump an object to a dictionary by calling the dump() method on a Schema class.

Once we have a dictionary of data for the meter reading, it’s time to add it to Redis.

First, we add it to the global feed -- all meter readings should appear in this feed:

    pipeline.xadd(global_key, serialized_meter_reading, maxlen=self.GLOBAL_MAX_FEED_LENGTH)

This method takes a pipeline object, and we're going to use that for our operations. Calling the xadd() method on the pipeline object queues an XADD Redis command to add the meter reading to the global feed identified by the global feed key.

Next we queue another XADD command to add the meter reading to the site feed:

    pipeline.xadd(site_key, serialized_meter_reading, maxlen=self.SITE_MAX_FEED_LENGTH)

In both cases, notice that we passed the maxlen argument to xadd(). This ensures that Redis trims the feed to our desired maximum length -- and recall that redis-py defaults to using Redis’s approximate trimming, for better performance.

At this point, your tests should pass -- and you’re done! After running make load, you should be able to see the global stream of events by visiting the app and clicking on the "Recent" menu item at the top of the page.
