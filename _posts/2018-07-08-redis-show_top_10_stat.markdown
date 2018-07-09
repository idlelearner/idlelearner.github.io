---
layout: post
title:  "Redis - show top 10 stat in 24 hrs"
categories: [Cache, Redis, Heap]
tags: [Design, cache, Redis, Development]
date:   2018-07-07 08:44:21 -0700
---

Redis is an open source, in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries.

Let's take an use case where we would need to get the top 10 stat every hour/24 hrs. Here the stat could be the top accessed attribute in the site like most searched phone number in whitepages.

Redis is super fast and has a rich set of data structures with functionalities. Here we can use the sorted set to build the heap. More documentation can be found in [Redis SortedSet](https://redis.io/commands/zadd). For the example, We could bucketize the phone numbers and their access count per hour. So we can keep creating new buckets every hour and use that as a sliding window of top n stat for the last 24 or 48 hrs.

**Design:**   
We have split the design into two components. First one will the write API which will record the stat. Second will the API which is responsible to read the top 10 stat from redis and return it. Lets see the write, read components in detail below.
  
**Write:**   
This would involve adding the stat and increment the access count in the current hour bucket(key : cur_date_hour). Here we also set the expiry for the sorted set to the required time so redis handles the deletion of this key and we don’t need to worry about the clean up.
{% highlight redis %}
ZADD 20180708_20 1 "XXX-XXX-XXXX" INCR EX 179200
{% endhighlight %}
Here I am setting the expiry time as 50 hrs. These sortedset will be expired after 50 hrs after the last write. Incrementing the count of an element can also be done using `ZINCRBY` but it does not have the expiry option. In that case we need to call `EXPIRE` for the key in a separate call which adds extra latency.  

**Read:**   
For reading the top 10 stat for the last 24 hrs, we combine the sorted set bucket starting from last hour to 24 hrs back. This can be achieved by using `ZUNIONSTORE`. So the sequence of steps hee would be to query for the key `top_cur_date_hr` and if the key exists, we can do `ZRANGE top_cur_date_cur_hr -10 -1`. This will give the top 10 stat in the sortedset. Here, -1 is the highest score and -10 is the 10th highest score. Some optimizations that can be done here are before calling `ZUNIONSTORE` we can remove the elements from the buckets that is not needed. If we are interested only in top 10 stat, then we can remove any stat whose rank (from bottom) greater than 240 as that will not make it to the top 10. Also note, by default the rank in redis based on lowest score first. We can call `ZREMRANGEBYRANK myzset 0 -241` to remove any stat which does not fall in top 120 scores.

Below are the steps before fetching the top 10 for the last 24 hrs.

{% highlight redis %}
ZRANGE top_20180708_20 -10 -1
{% endhighlight %}

If the above returns value, we return the top 10 stat. But if `err no such key` is returned which will happen when the new hour rolls, then we do the following
We get the cardinality of the set for the previous hour - Complexity `O(1)` - constant time
{% highlight redis %}
ZCARD 20180708_19
{% endhighlight %}

If the cardinality is greater than 240, then we remove the elements that are not needed. Complexity here is `O(log(N)+M)` with N being the number of elements in the sorted set and M the number of elements removed by the operation.
{% highlight redis %}
ZREMRANGEBYRANK 20180708_19 0 -241
{% endhighlight %}

Then we union the last 24 hrs sortedsets using `ZUNIONSTORE`. Complexity: `O(N)+O(M log(M))` with N being the sum of the sizes of the input sorted sets, and M being the number of elements in the resulting sorted set.
{% highlight redis %}
ZUNIONSTORE top_20180708_20 24 20180708_19 20180708_18 .. 20180707_20
{% endhighlight %}
The performance of the `ZUNIONSTORE` depends on the size of the sorted sets that are used to combine. For the last 24 hrs, the final size of the sorted set will be 5760 if all the elements in those sets are distinct. `ZUNIONSTORE` will just sum the counts when aggregating by default. Other aggregations can also be used while combining the sortedsets.

To further optimize, we can delete the elements which are not in top 10
{% highlight redis %}
ZREMRANGEBYRANK 20180708_19 0 -11
{% endhighlight %}

We also need to set the `EXPIRE` for the top_20180708_20 bucket.

For fetching the top 10 stat for the hour, we just call below.
{% highlight redis %}
ZRANGE top_20180708_20 -10 -1
{% endhighlight %}

I have discussed a way to build a heap in redis and use that to get top 10 attribute based on a statistics . Some of the things considered in this blog are keeping the entire design to one system which is redis here. This could be altered to use a job which runs every hour to do the compaction of the sorted sets. Above design should return results in the order of `ms` (just the first call in a hour could take a couple of `ms` but the subsequent calls would be constant time). Another assumption, we have made is, that there will always be a write to redis every hour. If that is not the case, then we might need to go back further while removing the elements which does not fall in top 240. This system can be extended for different use cases and different time limits.
