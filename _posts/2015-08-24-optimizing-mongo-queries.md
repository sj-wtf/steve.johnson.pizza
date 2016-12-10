---
layout: post
title: Optimizing Mongo Queries
description:
headline: Taming the hellish beast
modified: 2016-08-24
category: tech
tags: [update, java, jvm]
image:
---
A coworker of mine maintains that optimizing a query is one of the most enjoyable things in life.  I have to say, the feeling you get when you get a query down to the lowest possible response time is pretty great.  If the number of individual documents/rows that were checked to see if they matched the criteria of the query is the same as the number of documents that were returned by your query, you win!  This continues to hold true for MongoDB.  At any scale, it’s very important that your queries are efficient, however the harm caused by inefficient queries is much greater the larger you get.  The more documents you have to go through to find a match, the worse your query efficiency is, and the longer it can take to return the requested data.  This can get pretty bad over large collections.  I’ve seen it where a single query can take 45 minutes to complete due to the sheer number of documents scanned, even when backed by some *massively* (750MB/s 4k random reads) performant SSDs.  So, if you want your queries to take a reasonable amount of time, then index!


Normal Mongo indexes are b-tree indexes.  This is a common type of index in many types of databases out there.  They work by creating a tree structure that a cursor can traverse to find the document or group of documents that the query is looking for (optimal), or a group of documents that a cursor scans to find matches (sub-optimal, but sometimes OK), rather than going through every single actual record and looking for a match.  Each level of the tree narrows the number of documents that a query can possibly match by matching part of the query criteria.  Once you hit the end of the tree, you’re pointed at a document, or group of documents to scan.  The obviously better result is the exact documents that you want to return, and when possible, you should craft your index to do that.  One exception to this is if you’re running short on memory.  There are some techniques you can use to marginally decrease the amount of memory used by your indexes in some cases, with only a marginal decrease in performance.  I’ll talk about that a bit later.

You can see what indexes are currently available to you when querying a collection using Mongo’s JS shell, and running the query `db.<collection_name>.getIndexes()`. This should return something looking like this:

```
{
  "v":1,
  "key" : {
    "_id" : 1,
    "version.lastUpdated" : -1
  },
  "name" : "an_index",
  "ns" : "database_name.collection_name",
  "sparse" : false
}
```

Here’s a description of each field:

v: the version type of index that you want to use. Right now, Mongo has two index versions, 0 and 1. 0 is what existed in MongoDB 1.x up to 2.0. 1 is what’s used in 2.x and greater, including 3.

key: a document specifying the fields you want indexed in this index, and their sort orders. For example, `"_id"` : 1 means “index field `“_id”` with an ascending sort order”, and `"version.lastUpdated"` : -1 means “index field `“version.lastUpdated”` with a descending sort order”. As such, if you hint at an index, a sort order can be implied without explicitly sorting in your query. That being said, it’s still a much better idea to explicitly state a sort order in your query in case the execution planner doesn’t figure out that your query is supposed to use the index you specified.

name: the name of the index. Most database drivers allow you to refer to an index by name in their `cursor.hint()` method.

ns: the namespace that the index indexes. This consists of the database and the collection that the index applies to.

sparse: whether or not the index should consist of *all* documents, or just the most recent documents queried. This can make your indexes more space-efficient, but that space efficiency comes with a performance hit.

When working with very large datasets, it’s just as important to make sure that your sort order makes sense.  When your query specifies that you want your documents in a specific order, but your documents are returned in an arbitrary order, an in-memory sort is performed on the server side before returning your documents to you.  That’s a whole extra step that has to happen before you can continue, and it gets more expensive the more data you return!  Luckily, as I mentioned above, you can specify a sort order in your index.  If you do so, you can eek a bit more performance out of your queries which require a sort order by using the same order in your query as in your index.

One important aspect of being able to optimize a query is knowing your data.  Take this quasi real-world example which I have twisted slightly to suit my narrative.  A web portal application has a static query on 5 fields for each user login: “name”, “post_source”, “expiration_date”, “person_id”, and a sort on “date_received”.  This query is supposed to return a single document, which if it exists, skips displaying the terms of use.  If I made an index on all of those fields like I usually would, and it was 4082358560 bytes (about 4 gigs).  That’s 4 gigs of memory that we could otherwise keep data in for faster queries.  Knowing what the application is looking for, I could probably just make an index on person_id, name, and date_received sorted in the appropriate direction to avoid a find and order operation, and get similar performance!  I made the index on person_id, name, and date_received, and it was only 2377360048 bytes, a little more than half the size of the original index.  Pretty good, eh?  I’m not quite done, though.  Note how I said that this query returns the document, but the only use for the document is to feed what boils down to be a boolean.  A bit is much smaller than a bson document of any size.  So, I could just throw a .count() on the end of the query, if I’m using something like C/C++, which interprets any non-zero value to be “True” in a boolean, or just throw some code in to say something like “if the return value of this query is greater than 0, bool == false, else true”.  Presto, we’ve just reduced the number of bytes we’re sending over the wire by quite a lot!

Fun times with query optimization!
