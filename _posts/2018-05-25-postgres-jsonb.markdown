---
layout: post
title:  "JSONB - NoSQL in postgres"
date:   2018-05-25 08:44:21 -0700
---
We use postgres and different types of NoSQL systems in our production systems. There is always a question whether to go with either adding those values to the RDBMS, postgres in our case or simply go with NoSQL. NoSQL gives you the flexibility to not normalize the data and going schemaless. But there is a caveat when two different systems are used for persisting values. You need to take care of ACID in your application code. It will get harder as table gets complex.

Some examples of schemaless data you might want to store could be normalized address(address_line_1,address_line_2, city, state, zip code). If you don’t get all the address values and it is simpler to store address object as a column. Other examples are storing receipts, reports from other external APIs which can be persisted before processing, adding metadata/properties to an attribute.

So a better solution would be store these schemaless values as a column with other attributes in your table .That will save a lot of headache querying two different systems. Postgres has JSON support for a while but `postgres 9.4` added indexing which is the most important feature when it comes to performance in DB. JSONB will be ideal when you don't need extensive querying on the data stored inside it. Summarizing the pros and cons below.

**Pros:**
- Easier to store all the values in one system, database takes care of ACID property.
- Allows indexing - `GIN (Generalized Inverted Index) index`, making it faster to access.
- Ideal for schemaless where normalizing the data attributes would be overengineering.

**Cons:**
- JSONB is stored as strings so will take up more space in the table. It will be a concern when your DBA has to handle 100s of GBs of data when doing backup/migration.
- Postgres does not store metadata values about NULL values, counts and distinct. So query related to that might be slower.

You can also use jsonb_pretty to pretty print the JSON data

{% highlight SQL %}
-- pretty print JSONB
SELECT jsonb_pretty(json_data) FROM table;
{% endhighlight %}

We have been storing values as JSONB in our production and has saved a lot of development time and keeps the application code simple.

**References:**  
[www.postgresql.org/docs/9.4/static/datatype-json.html](https://www.postgresql.org/docs/9.4/static/datatype-json.html)  
[blog.codeship.com/unleash-the-power-of-storing-json-in-postgres/](https://blog.codeship.com/unleash-the-power-of-storing-json-in-postgres/)  
[www.sarahmei.com/blog/2013/11/11/why-you-should-never-use-mongodb/](http://www.sarahmei.com/blog/2013/11/11/why-you-should-never-use-mongodb/)  
[heapanalytics.com/blog/engineering/when-to-avoid-jsonb-in-a-postgresql-schema](https://heapanalytics.com/blog/engineering/when-to-avoid-jsonb-in-a-postgresql-schema)  



