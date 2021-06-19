Challenge #5 Review

In Challenge #5, we asked you to get the "Excess Capacity" filter working in the app. Here is a solution to that problem:

    def _find_by_geo_with_capacity(self, query: GeoQuery, **kwargs) -> Set[Site]:
        # START Challenge #5
        # Your task: Get the sites matching the GEO query.
        coord = query.coordinate
        site_ids = self.redis.georadius(  # type: ignore
            self.key_schema.site_geo_key(), coord.lng, coord.lat, query.radius,
            query.radius_unit.value)
        # END Challenge #5

        p = self.redis.pipeline(transaction=False)

        # START Challenge #5
        #
        # Your task: Populate a dictionary called "scores" whose keys are site
        # IDs and whose values are the site's capacity.
        #
        # Make sure to run any Redis commands against the Pipeline object
        # "p" for better performance.
        for site_id in site_ids:
            p.zscore(self.key_schema.capacity_ranking_key(), site_id)
        scores = dict(zip(site_ids, p.execute()))
        # END Challenge #5

        for site_id in site_ids:
            if scores[site_id] and scores[site_id] > CAPACITY_THRESHOLD:
                p.hgetall(self.key_schema.site_hash_key(site_id))
        site_hashes = p.execute()

        return {FlatSiteSchema().load(site) for site in site_hashes}

That’s a lot of code, so let's break it down by task, since you had two main tasks to complete.
Step 1: Get sites that match the geo query

Your first task was to get the sites that matched the geo query. This code does that:

    coord = query.coordinate
    site_ids = self.redis.georadius(  # type: ignore
        self.key_schema.site_geo_key(), coord.lng, coord.lat, query.radius,
        query.radius_unit.value)

Given a GeoQuery that contains coordinates near which to search, this code runs the GEORADIUS command, passing in the coordinates, radius, and the type of measurement to use for the radius -- all taken from the GeoQuery (which itself is derived from the search form on the front-end).

GEORADIUS returns all the IDs of sites within that radius, which completes Step 1.
Step 2: Build the "scores" dictionary

Once we have the IDs of the sites in the chosen radius, we need to create a dictionary whose keys are the site IDs and values are the site capacities. Here’s how the solution code does this:

    for site_id in site_ids:
        p.zscore(self.key_schema.capacity_ranking_key(), site_id)
    scores = dict(zip(site_ids, p.execute()))

First, we loop over all the site IDs we got back from GEORADIUS, and we queue up a ZSCORE command for each on the Pipeline object p. Pipelines don’t return their results until you call execute(), so on the next line we do that.

This line packs a lot in just a few function calls, so let’s break it down:

scores = dict(zip(site_ids, p.execute()))

First, we call p.execute(). This returns all the site capacities in the same order as our site IDs list.

Consider that for any given site ID in site_ids, that site's capacity is in the same index position in the list returned by p.execute() -- because we get back the responses to our ZSCORE calls in the same order in which we made them.

For example, the site whose ID is in site_ids[10] has its capacity in p.execute()[10].

Now ultimately, we want to produce a dictionary with these values. And if we can somehow combine these two lists, we can pass them into the built-in dict() function to produce a dictionary...

Thanks to both the site_ids list and the result of p.execute() being in the same order, we can do this easily. The Python built-in zip() can combine multiple iterables. So, we call zip(site_ids, p.execute()) to produce a sequence of tuples -- each tuple containing a site ID and the site's capacity.

You can imagine it like this. Before, we had two lists:

First, the site IDs, which looked like this:

    [1, 2, 55, 8]

Then the capacities from p.execute():

    [45, 114, 97, 88]

In order to use the dict() function, we need to end up with these lists combined, like this:

    [[1, 45], [2, 114], [55, 97], [8, 88]]

That’s the structure that zip(site_ids, p.execute()) produces, so we can pass its return value directly into dict() to create a dictionary that looks like this:

    {
    1: 45,
    2: 114,
    55: 97,
    8: 88
    }

In other words, a dictionary whose keys are the site IDs, and whose values are the capacities of those sites.

And that’ll solve it!

You may or may not have known about zip() and dict(), but don’t worry -- how you produced this dictionary doesn’t really matter as long as you managed to produce a dictionary with the right structure!