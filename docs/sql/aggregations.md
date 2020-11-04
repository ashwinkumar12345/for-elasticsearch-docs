---
layout: default
title: Aggregation Functions
parent: SQL
nav_order: 50
---

# Aggregation functions

Aggregate functions use the `GROUP BY` clause to group sets of values into subsets.

## Group By

Use the `GROUP BY` as an identifier, ordinal, or expression.

### Identifier

```sql
SELECT gender, sum(age) FROM accounts GROUP BY gender;
```

| gender | sum (age)
:--- | :---
F | 28 |
M | 101 |

### Ordinal

```sql
SELECT gender, sum(age) FROM accounts GROUP BY 1;
```

| gender | sum (age)
:--- | :---
F | 28 |
M | 101 |

### Expression

```sql
SELECT abs(account_number), sum(age) FROM accounts GROUP BY abs(account_number);
```

| abs(account_number) | sum (age)
:--- | :---
| 1  | 32  |
| 13 | 28  |
| 18 | 33  |
| 6  | 36  |

## Aggregation

Use aggregations in select, expressions, or arguments of expressions.

### Select

```sql
SELECT gender, sum(age) FROM accounts GROUP BY gender;
```

| gender | sum (age)
:--- | :---
F | 28 |
M | 101 |

### Arguments of expression

The aggregation could be used as arguments of expression:

```sql
SELECT gender, sum(age) * 2 as sum2 FROM accounts GROUP BY gender;
```

| gender | sum2
:--- | :---
F | 56 |
M | 202 |

### Expression as an argument

```sql
SELECT gender, sum(age * 2) as sum2 FROM accounts GROUP BY gender;
```

| gender | sum2
:--- | :---
F | 56 |
M | 202 |

### COUNT

Use the `COUNT` function to accepts arguments such as * or literals like 1.
The meaning of these different forms are as follows:

- `COUNT(field)` - Only counts if given a field (or expression) is not null or missing in the input rows.
- `COUNT(*)` - Counts the number of all its input rows.
- `COUNT(1)` (same as `COUNT(*)`) - Counts any non-null literal.

## Having

Use the `HAVING` clause to 
A HAVING clause can serve as aggregation filter that filters out aggregated values satisfy the condition expression given.

### HAVING with GROUP BY

Aggregate expressions or its alias defined in SELECT clause can be used in HAVING condition.

    It's recommended to use non-aggregate expression in WHERE although it's allowed to do this in HAVING clause.
    The aggregation in HAVING clause is not necessarily same as that on select list. As extension to SQL standard, it's also not restricted to involve identifiers only on group by list.

Here is an example for typical use of HAVING clause:

```sql
SELECT gender, sum(age)
FROM accounts
GROUP BY gender
HAVING sum(age) > 100;
```

| gender | sum (age)
:--- | :---
M | 101 |

Here is another example for using alias in HAVING condition. Note that if an identifier is ambiguous, for example present both as a select alias and an index field, preference is alias. This means the identifier will be replaced by expression aliased in SELECT clause:

```sql
SELECT gender, sum(age) AS s
FROM accounts
GROUP BY gender
HAVING s > 100;
```

| gender | s
:--- | :---
M | 101 |

### HAVING without GROUP BY

Additionally, a HAVING clause can work without GROUP BY clause. This is useful because aggregation is not allowed to be present in WHERE clause:

```sql
SELECT 'Total of age > 100'
FROM accounts
HAVING sum(age) > 100;
```

| Total of age > 100 |
:--- |
Total of age > 100 |
