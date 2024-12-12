## Concatenation in Postgres using ||  

```sql
SELECT 
    'abc' || 12 as alpha,
    ARRAY[ROW(1,2,3)] || ARRAY[ROW(3,5,6)] as arrayWithRow, 
    ARRAY[1,2,3] || ARRAY[3,5,6] as arrayWithoutRow
```
 
| alpha | arrayWithRow | arrayWithoutRow |
| -- | --- | -- |
| abc12 | {"(1,2,3)","(3,5,6)"} | {1,2,3,3,5,6} |

 ## Data Type Casting in Postgres using Cast() or :: operator

 `SELECT  CAST ('100' AS INTEGER);`  
 `SELECT '2019-06-15 14:30:20'::timestamp;`  
[![image](https://github.com/user-attachments/assets/f70fe4c6-6ad0-4a82-96e2-b7f68b70dece)](https://neon.tech/postgresql/postgresql-tutorial/postgresql-cast)


