Challenge #4 Review

In Challenge #4, we asked you to implement the get_rank() method for the CapacityReportDaoRedis class. Here is a solution to that problem:

    def get_rank(self, site_id: int, **kwargs) -> float:
        capacity_ranking_key = self.key_schema.capacity_ranking_key()
        return self.redis.zrevrank(capacity_ranking_key, site_id)

Let’s walk through how this works.

First, we get the key that stores capacity rankings:

capacity_ranking_key = self.key_schema.capacity_ranking_key()

You can see this key is used elsewhere in the class, like its update() method.

Once we have the key, we’re going to run the ZREVRANK command to get the rank of the site in descending order -- that is, ordered highest to lowest.

Recall that this is a Sorted Set, and we’re sorting on the site's current capacity. So the #1 site in reverse order is the site with the most capacity.