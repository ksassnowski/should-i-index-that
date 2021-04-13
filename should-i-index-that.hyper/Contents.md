# Should I index that?


:: size: 140
It depends.

---
# Origin of this talk

![](CleanShot_2021-04-01_at_13.57.21@2x.png)

--- :: number: true
# The responses

- Does it make sense to do … ?
- When should I do … ?
- How do I optimize this query?

--- ::number: true
# It depends!

--- ::number: true
:: size: 140, label: How do I index this query?, theme: InspiredGitHub
```sql
 SELECT * ❶
   FROM table ❶
  WHERE column_1 >= 7 ❷
    AND column_2 >= 3; ❸
```

--- ::number: true, master: regular1

---
:: size: 140, theme: InspiredGitHub
```sql
SELECT *
  FROM table 
 WHERE column_1 >= 7 
   AND column_2 >= 3; 
```


### Observations

:: size: 100
- Both of the conditions use `>=` as the comparison operator.
- This means we’re dealing with _ranges without an upper bound_.

--- ::alignment: left, master: regular1, number: true