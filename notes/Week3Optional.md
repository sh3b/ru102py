In the Week 3 bonus challenge, we gave you an optional challenge: make the find_all() method from the SiteGeoDaoRedis class use pipelines for its calls to self.redis.hgetall().

The following is a solution to that challenge:

    def find_all(self, **kwargs) -> Set[Site]:
        """Find all Sites."""
        site_ids = self.redis.zrange(self.key_schema.site_geo_key(), 0, -1)
        sites = set()
        p = self.redis.pipeline(transaction=False)

        for site_id in site_ids:
            p.hgetall(self.key_schema.site_hash_key(site_id))

        site_hashes = p.execute()

        for site_hash in [h for h in site_hashes if h is not None]:
            site_model = FlatSiteSchema().load(site_hash)
            sites.add(site_model)

        return sites

Let’s walk through how this code works.

First, we create a Pipeline object:

p = self.redis.pipeline(transaction=False)

Recall that with redis-py, you pass the argument transaction=False to the pipeline() method to use a true Redis pipeline instead of running your commands in a transaction.

Then we get all of the site hashes using the pipeline we just created:

for site_id in site_ids:
    p.hgetall(self.key_schema.site_hash_key(site_id))

site_hashes = p.execute()

Before, we ran HGETALL for each site, but now we buffer all those HGETALL commands into a pipeline and run them together with a single request to Redis.

Because we have all the hashes, the rest of the method can focus on loading them into our Site models using the FlatSiteSchema class:

for site_hash in [h for h in site_hashes if h is not None]:
    site_model = FlatSiteSchema().load(site_hash)
    sites.add(site_model)

return sites

And we’re done!