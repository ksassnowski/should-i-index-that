:: size: 160
# Hi, _Laravel Worldwide_!

---
:: size: 170
# How should I _index_ this _query_?

--- :: number: true
:: size: 300
# It _depends_

--- :: number: true
:: size: 270
# Thanks!

**Twitter** 
@warsh33p
**Web**
https://kai-sassnowski.com
**GitHub** 
ksassnowski

--- :: number: true, master: regular1

# Origin of this talk

![](CleanShot_2021-04-01_at_13.57.21@2x.png)

--- :: number: true
- Does it make sense to do … ?
- When should I do … ?
- How do I optimize this query?

# The responses

:: size: 70
![](output-onlinepngtools.png)

--- ::number: true, alignment: right
**Side note**
All examples assume that there are no existing indices.

--- :: number: true, master: regular1
:: size: 60
![](CleanShot_2021-04-13_at_22.png)
# Scenario #1
:: size: 80
## This looks pretty harmless... right?

--- ::number: true, alignment: right
:: size: 130
## The query

:: size: 140, theme: InspiredGitHub
```sql
 SELECT *
   FROM table
  WHERE column_1 >= 7; ❷
```

--- ::number: true, master: regular2
# Vote now!
## https://www.strawpoll.me/42936763

--- ::number: true
![website](https://www.strawpoll.me/42936763)

--- ::number: true, master: regular2
# The correct answer was...

*** :: master: title2, number: true
:: size: 50

_(pause for dramatic effect)_

--- :: number: true
:: size: 170
# ✅  I don't know

--- ::number: true, master: regular1
# Things I forgot to tell you

1. The table has about 6 million rows
2. `column_1` contains only numbers between `1` and `9`
3. The data looks like this:

--- :: number: true, master: title1
:: size: 85
![](output-onlinepngtools_(1).png)

--- :: number: true, master: regular1, transition: (0.50;fade)
![](output-onlinepngtools_(1).png)

## Observations

- The data is **not** evenly distributed
- Almost all of the data clusters around the 7-9 range.

--- :: number: true, alignment: left, master: regular1
:: size: 160
# So what?

--- :: number: true, master: title1
:: size: 140, theme: InspiredGitHub
```sql
 SELECT *
   FROM table
  WHERE column_1 >= ?;
```


:: animate: (0.30;fade)
What’s the **minimum** number of rows this query can select?

--- :: master: regular2, number: true
:: size: 120, theme: InspiredGitHub
```sql
SELECT COUNT(*)
  FROM table
 WHERE column_1 >= 9;
```

:: size: 120
```
count(*): 1932891 ❶
```

*** :: master: regular2, alignment: left, number: true
:: size: 100
![](CleanShot_2021-04-14_at_17.png)

:: size: 150
# Well **then...**

--- :: number: true, alignment: right, master: regular1
:: size: 130
# Indices _don't work well_ for queries that return _lots of data_

--- ::number: true, master: title1
:: size: 110
# A really quick explanation[^1]

- An index only contains data from **the indexed column(s)**
- Any additional data needs to be **fetched from the table**
    - like in `SELECT *`
    - ...for **every row** that is matched

[^1]: https://www.youtube.com/watch?v=HubezKbFL7E

--- ::number: true, master: regular1
:: size: 200
## So, how _should_ I index this query then?

--- ::number: true, master: regular1
:: size: 280
# You _don't_!

--- :: number: true, master: regular1
:: size: 180
## An index _does not_ make a query _more selective_
--- ::number: true, master: regular1
:: size: 140 
## Exercise for the ~~reader~~ listener

How would the answer change if the distribution was skewed the other way?
--- ::number: true
# Scenario #2
## To the moon 📈

--- ::number: true, master: title2
# Some context

- You’re working on an **e-commerce** platform
- Many sellers, many buyers
- All orders get saved in **same table**
    - `orders` table can easily have **hundreds of millions** of rows

--- ::number: true
## The query

:: size: 88, theme: InspiredGitHub
```sql
WITH t AS ( ❹
	  SELECT DATE(created_at) AS d, ❶
	         SUM(total) AS total_order_sum ❶
	    FROM orders ❶
	   WHERE created_at BETWEEN '2021-03-01T00:00:00' AND '2021-03-31T23:59:59'❷
 GROUP BY d ❸
) ❹
   SELECT d, ❺
  		      total_order_sum, ❺
          AVG(total) OVER w AS a ❺
     FROM t ❺
   WINDOW w AS (ORDER BY d DESC RANGE INTERVAL 3 DAY PRECEDING) ❻
 ORDER BY d DESC; ❼
```
--- :: number: true, master: regular2
:: size: 140
_Question_

:: size: 150
# **How** is this query **used**?

--- ::number: true
:: size: 140
_Question_

:: size: 150
# **How often** does this query run?

--- ::number: true
:: size: 75
![](output-onlinepngtools_1.png)


:: size: 180
## **Case #1**

Query is used to generate a report that gets sent to each seller at the beginning of each month.


--- ::number: true, alignment: left, master: regular1

:: size: 75
![](output-onlinepngtools_1.png)


:: size: 180
## **Case #1**

Query is used to **generate a report** that gets **sent to each seller** at the **beginning of each month**.

--- ::number: true, master: regular1, alignment: left
:: size: 180
## **Case #1**

- Query is only used to **generate an automated report**
    - **not triggered** by user interaction
    - performance **doesn’t matter**
- Query runs **once a month** at night
- `orders` table is **very large** (potentially hundreds of millions of rows)
  

--- ::number: true, master: regular1
# **Case #1**
:: size: 160
# Solution: **Don't index**

--- ::number: true, master: regular1
- The index would be **really large**
- It would only be used **once a month**
    - For a query where **performance doesn't matter**
- It would **affect write performance** on the `orders` table **all the time**.

:: size: 80
# **Case #1**
:: size: 80
# Solution: **Don't index**

--- ::number: true, master: regular1, alignment: right
## Alternative solutions

- Spread report generation over multiple hours
- Pre-compute metric during the month

--- ::number: true , master: regular1
image

:: size: 180
## **Case #2**

Query is used to display a real-time chart on the seller’s dashboard page.

--- ::number: true, master: regular1, alignment: right
image

:: size: 180
## **Case #2**

Query is used to display a **real-time chart** on the **seller’s dashboard page**.

--- :: number: true, master: regular1, alignment: right
:: size: 180
## **Case #2**

- Query is triggered by **user interaction**
- Query is used to display **real time data**
- Runs **dozens** if not **hundreds** of times **per second**

--- ::number: true, master: regular1
# **Case #2**

:: size: 140
# Solution: **It’s complicated**

--- :: master: regular1, number: true
- Index aggressively (covering index)
- Cache results (does it _have_ to be real-time?)
- Pre-compute results

:: size: 80
# **Case #2**

:: size:80
# Solution: **It’s complicated**

--- ::master: regular1, number: true, alignment: right
:: size: 80
![](output-onlinepngtools.png)

--- :: alignment: vertical, master: regular1, number: true
# To sum up

- Indices **don’t exist in a vacuum**
- Not every slow query can be improved by adding an index
- Developers need to know this stuff

--- :: number: true
:: size: 270
# Context is **everything**

--- ::number: true
:: size: 270
# Thanks!

:: size: 50
_(For real this time)_

**Twitter** 
@warsh33p
**Web**
https://kai-sassnowski.com
**GitHub** 
ksassnowski


*** :: master: regular1