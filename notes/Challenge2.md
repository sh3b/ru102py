Challenge #2 Review

In Challenge #2, you completed the __insert_metric()__ method of the MetricDaoRedis class.

Here is one solution to that challenge:

# Challenge #2

    def insert_metric(self, site_id: int, value: float, unit: MetricUnit,
                      time: datetime.datetime, pipeline: redis.client.Pipeline):
        metric_key = self.key_schema.day_metric_key(site_id, unit, time)
        minute_of_day = self._get_day_minute(time)

        pipeline.zadd(metric_key,
                    {str(MeasurementMinute(value, minute_of_day)): minute_of_day})
        pipeline.expire(metric_key, METRIC_EXPIRATION_SECONDS)

You can find a copy of this solution in the "solutions" branch in the course GitHub repo, or in the "solutions" folder in the Docker container.
How Does it Work?

So, how does this solution work?

We gave you the first two lines, which create the Redis key to use for the new metric and find the minute of the day in which the metric occurred.

In our solution, we do a few things in a single line:

    pipeline.zadd(metric_key, {str(MeasurementMinute(value, minute_of_day)): minute_of_day})

The first is to create an instance of MeasurementMinute and immediately convert it to a string:

    str(MeasurementMinute(value, minute_of_day))

This produces the specially-formatted string value that we need to store our metric: <value>:<minute_of_day>.

To add this value to Redis, we’re calling __pipeline.zadd()__, which takes the Redis key to use -- in this case the value of metric_key -- and a dictionary.

The way the dictionary parameter works might be confusing, so bear with me! The key of the dictionary is the value of the sorted set member (<value>:<minute_of_day>), while the value of the dictionary is the score of the sorted set member. We use the minute of the day as the score.

Next, we use __expire()__ to remove the sorted set after a configured amount of time because we assume the application won’t need it after then.

Finally, note that we used a pipeline to send both of these commands to Redis in a single round trip.
