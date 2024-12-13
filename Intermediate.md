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

## Array Indexing

- Arrays in Postgres are 1-based. `season[1]` will output the 1st element of an array named `seasons`.
- `CARDINALITY()` is an array function that counts the total number of array elements. so `season[CARDINALITY(season)]` will output the _rightmost_ element of an array.
- todo: For 2d array or multidimensional array, [see this](https://www.commandprompt.com/education/how-to-use-cardinality-function-in-postgresql/#:~:text=Example%202%3A%20CARDINALITY()%20Function%20With%20Multi%2DDimensional%20Array)

## Order by 2, 1, 3
- Generally we specify a 'column name' after `ORDER BY`. We can also use numbers, e.g. `ORDER BY 2 DESC` will sort the 2nd column.  
- ![image](https://github.com/user-attachments/assets/12cf3649-452e-4b72-9b1d-b3e4bb217d56)
### INSERT INTO vs INSERT OVERWRITE vs MERGE

