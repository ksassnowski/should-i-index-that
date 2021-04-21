# Should I index that?


:: size: 140
It depends.

---
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

# Well then...

--- :: number: true, alignment: right, master: regular2
# Indices _don't work well_ for queries that return _lots of data_

--- ::number: true
# So how _should_ I index this query?

--- ::number: true, master: regular1
:: size: 280
# You _don't_!

--- :: number: true, master: regular1
:: size: 180
## An index _does not_ make a query _more selective_
--- ::number: true
# Scenario #2
## To the moon 📈

--- ::number: true, master: title2
## The query

:: size: 88, theme: InspiredGitHub
```sql
WITH t AS ( ❹
	  SELECT DATE(created_at) AS d, ❶
	         SUM(total_disk) AS total ❶
	    FROM computers ❶
	   WHERE created_at BETWEEN '2021-03-01T00:00:00' AND '2021-03-31T23:59:59'❷
 GROUP BY d ❸
) ❹
   SELECT d, ❺
  		      total, ❺
          AVG(total) OVER w AS a ❺
     FROM t ❺
   WINDOW w AS (ORDER BY d DESC RANGE INTERVAL 3 DAY PRECEDING) ❻
 ORDER BY d DESC; ❼
```
--- :: number: true, master: regular2
:: size: 300
# 😅
--- :: number: true
