---
title: "In search of a faster query"
categories:
  - Rails
  - MySQL
---

Recently, as I was fixing a minor bug in one of our applications at work, I had the opportunity to witness what a great difference a simple query can make.

The bug involved a very slow query, that was doing a join between a couple of tables that had grown too large in size. The query, in Rails, was as follows:

```ruby
Model1.joins(:model2).order(:attribute1)
```

That query alone was taking nearly 2200 ms to complete,  kind of showing that it really is the devil to make a join between two large tables. The table for Model1, for instance, had nearly 90,000 rows and 10+ columns, while the table for Model2 was slightly smaller, with nearly 30,000 rows.

After examining the code, I realized we really only needed the id and attribute1 of Model1 records, so I modified the query slightly to retrieve only that:

```ruby
Model1.select('attribute1, model1.id').joins(:model2).order(:attribute1)
```

Wow! This simple change brought the time down to around 520 ms, which was still sluggish, but a great improvement from the 2200 ms we began with.

It was a third iteration of the query, however, that  made the most impact. We had a third table, for Model3,  with an association to Model1  that we could use to our advantage. Although with no association between them, Model2 and Model3 both had a model1_id column, allowing us to do something like this:

```ruby
model1_ids = Model3.joins('INNER JOIN model2 on model2.model1_id = model3.model1_id')
                   .uniq.pluck('model3.model1_id')
Model1.where(id: model1_ids).order(:attribute1)
```

That last query brought the time down to around 40 ms. Not bad at all!

I felt very pleased by this results, mostly because I learned  it pays to play around with different ways to do the same, and to keep an eye on the query times reported in the console. It’s also worth thinking about how large the tables are likely to grow, although I know it’s hard to predict the future. It is possible when this part of the application was written, some time ago, that first query used to be fast enough, most likely because the table was small too.
