# DECISIONS.md

1. **Indexes.** 
I created a unique index on `employees.emp_code` to enforce uniqueness and optimize `find_one` lookups during punch-ins/outs. I created a compound unique index on `attendance_logs(emp_code, date)`. This index serves three purposes: it enforces the "one record per day per employee" rule, it prevents race conditions via `DuplicateKeyError`, and it optimizes the `GET /attendance` query which sorts by `date` then `emp_code`.

2. **Punch-in race.** 
If two identical punch-in requests arrive at the exact same instant, both might pass the `find_one` check. However, they will both attempt to execute `insert_one(doc)`. Because of the compound unique index on `(emp_code, date)`, MongoDB guarantees atomicity at the document level. One request will succeed and get a 201 Created. The second will throw a `DuplicateKeyError`, which the API catches and translates into a 409 Conflict.

3. **Ties.** 
If two employees tie on late minutes at the cutoff limit, my leaderboard returns both of them, meaning the total returned rows can exceed the limit. This is because I used the MongoDB window function `$rank`, which implements standard competition ranking (e.g., 1, 2, 2, 4), and applied the `$match` filter (`rank <= limit`) *after* the ranking stage, as required by the business rules.

4. **Headcount.** 
The department summary counts employees with zero logs by starting the aggregation pipeline on the `employees` collection (matching by department and `joined_on <= end_date`) and doing a `$lookup` into the `attendance_logs` collection. By using `$unwind` with `preserveNullAndEmptyArrays: true`, employees without logs remain in the pipeline, allowing them to be added to the `$addToSet` headcount logic

5. **One thing you would change** 
If serving 100x the data (~10 million records), calculating rolling 7-day averages on the fly using `$setWindowFields` and `$densify` on every request would become too slow. I would change the architecture to pre-calculate and materialize daily department statistics into a separate `daily__department_aggregates` collection using a nightly cron job or MongoDB trigger  .