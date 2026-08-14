# SQL — Interview Practice Questions

## Basic (1-10)

**1. Find employees with salary > 50000**
```sql
SELECT * FROM employees WHERE salary > 50000;
```

**2. Find the second highest salary**
```sql
SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);
-- Or using LIMIT:
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;
```

**3. Find duplicate emails in a table**
```sql
SELECT email FROM users GROUP BY email HAVING COUNT(email) > 1;
```

**4. Count employees in each department**
```sql
SELECT department_id, COUNT(*) FROM employees GROUP BY department_id;
```

**5. Find employees who joined in the last 30 days**
```sql
SELECT * FROM employees WHERE join_date >= CURRENT_DATE - INTERVAL '30 days';
```

**6. Delete duplicate rows keeping one**
```sql
DELETE FROM users WHERE id NOT IN (SELECT MIN(id) FROM users GROUP BY email);
```

**7. Find the nth highest salary using window functions**
```sql
WITH RankedSalaries AS (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank FROM employees
)
SELECT salary FROM RankedSalaries WHERE rank = N;
```

**8. Self join — find employees and their managers**
```sql
SELECT e.name as Employee, m.name as Manager
FROM employees e LEFT JOIN employees m ON e.manager_id = m.id;
```

**9. LEFT JOIN to find customers with no orders**
```sql
SELECT c.name FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;
```

**10. UNION vs UNION ALL example**
```sql
-- UNION removes duplicates, UNION ALL keeps them.
SELECT name FROM employees
UNION
SELECT name FROM contractors;
```

## Intermediate (11-20)
11. Running total using window functions (`SUM(amount) OVER (ORDER BY date)`)
12. Rank employees by salary within department (`RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC)`)
13. Find consecutive login days for each user
14. Pivot table query (rows to columns)
15. Find the department with the highest average salary
16. CTE — recursive query for org hierarchy
17. Find products that were never ordered
18. Moving average using window functions
19. Gap and islands problem
20. Correlated subquery — find above-average salary per department

## Advanced (21-30)
21. Median salary per department
22. Year-over-year growth calculation
23. Session analysis — find user sessions with gaps
24. Find the longest consecutive sequence
25. Cross join for date dimension generation
26. Dynamic ranking with ties handling
27. Tree structure queries (adjacency list to path)
28. Time-series analysis with LAG/LEAD
29. Optimizing a slow query (explain the strategy - EXPLAIN ANALYZE, indexing)
30. Write a query to implement pagination efficiently (Keyset pagination vs OFFSET)
