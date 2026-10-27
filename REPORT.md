# Hands-on L10: Report

**Name: Lauren Sharpe**
**Student ID: 801371099**
**Email: lsharp22@charlotte.edu**

---

## Seed and commands

Seed used for `datagen.py`:

The commands you ran, in order. If you deviated from the steps in the README, say where and
why.

```bash
python3 datagen.py 8001371099
docker compose up -d
docker cp main.py spark-master:/opt/spark/work-dir/
docker exec -it spark-master /opt/spark/bin/spark-submit --master spark://spark-master:7077 /opt/spark/work-dir/main.py task1 /opt/spark/work-dir/shared
docker cp main.py spark-master:/opt/spark/work-dir/
docker exec -it spark-master /opt/spark/bin/spark-submit --master spark://spark-master:7077 /opt/spark/work-dir/main.py task1 /opt/spark/work-dir/shared
docker exec -it spark-master /opt/spark/bin/spark-submit --master spark://spark-master:7077 /opt/spark/work-dir/main.py task2 /opt/spark/work-dir/shared update

```

---

## Results

### Task 1: ingest and parse

Paste the micro-batch summary printed at the end of the run, and the number of rows in
`shared-folder/output/task1` (`cat shared-folder/output/task1/*.csv | grep -vc trip_id`).

```
=== task1: micro-batch summary ===
batch input rows            watermark state rows
    0         21                    -          -
    1         24                    -          -
    2         27                    -          -
    3         25                    -          -
    4         24                    -          -
    5         26                    -          -
    6         25                    -          -
    7         25                    -          -
    8         21                    -          -
    9         22                    -          -
   10         23                    -          -
   11         23                    -          -
```
286 rows
How many micro-batches ran, and why that number?
12 because each file is processed per batch and theres 12 files. 

### Task 2: trips and fares per window

Paste the windows written by the stream (`cat shared-folder/output/task2/*.csv | grep -v
window_start | sort`), the summary, and the batch table that `main.py` prints at the end.

```
=== task2: the same windows computed in batch over all files ===
+-------------------+-------------------+-----+----------+
|window_start       |window_end         |trips|total_fare|
+-------------------+-------------------+-----+----------+
|2026-10-27 07:58:00|2026-10-27 08:00:00|5    |191.15    |
|2026-10-27 08:00:00|2026-10-27 08:02:00|49   |2139.73   |
|2026-10-27 08:02:00|2026-10-27 08:04:00|52   |2264.86   |
|2026-10-27 08:04:00|2026-10-27 08:06:00|51   |2055.83   |
|2026-10-27 08:06:00|2026-10-27 08:08:00|54   |2170.25   |
|2026-10-27 08:08:00|2026-10-27 08:10:00|40   |1611.68   |
|2026-10-27 08:10:00|2026-10-27 08:12:00|35   |1634.9    |
+-------------------+-------------------+-----+----------+
=== task2: micro-batch summary ===
batch input rows            watermark state rows
    0         21                    -          2
    1         24  2026-10-27 07:58:55          2
    2         27  2026-10-27 07:59:46          3
    3         25  2026-10-27 08:00:46          2
    4         24  2026-10-27 08:01:55          3
    5         26  2026-10-27 08:02:56          2
    6         25  2026-10-27 08:03:53          3
    7         25  2026-10-27 08:04:51          2
    8         21  2026-10-27 08:05:56          3
    9         22  2026-10-27 08:06:47          2
   10         23  2026-10-27 08:07:41          3
   11         23  2026-10-27 08:08:50          2
   12          0  2026-10-27 08:09:51          2
```

---

## Questions

**1. Append versus update.** Run task 2 a second time with `update` (it prints to the
console). In which batch does each window first appear in update mode, and in which batch
is it written in append mode? Explain the difference with the watermark values from the
summary.
In update mode, Spark printed a window whenever its aggregate changed. In append mode Sparked waited until the watermark passed the end of the window before writing it.


**2. Where did the rows go?** Compare the stream's windows with the batch table. Which
windows have fewer trips in the stream, by how many, and why? Which windows are missing
from the stream's output altogether, and what would make Spark write them? (Look at
`datagen.py` for how the late events were made.)
The 8:00-8:02 window had 47 trips in the stream compared to 49 in the batch which is 2 fewer. 8:02-8:04 had 51 trips in the stream compared to 52 in the batch which is 1 fewwer. This is because the batch job counted every file together without a watermark, while the streaming job could discard very late events after the watermark had passed. 8:08-8:10 and 8:10-8:12 are missing because the final watermark happend before. If later events arrived with sufficiently newer event times, they could advanve the watermark and Spark would write the windows. 



**3. Checkpoint.** Run task 1 again without deleting `shared-folder/checkpoint/task1`. What
happens, and why? List the subfolders of the checkpoint and say in one line what each one
holds.
=== task1: micro-batch summary ===
batch input rows            watermark state rows
   12          0                    -          -

It doesn't reprocess the files again since it recorded which ones were alreadly processed. 


---

## Problems and fixes

Anything that went wrong and what resolved it. Paste the actual error message. If nothing
went wrong, say so.
[UNRESOLVED_COLUMN.WITH_SUGGESTION] A column, variable, or function parameter with name `drivier_id` cannot be resolved. Did you mean one of the following? [`driver_id`, `trip_id`, `event_time`, `distance_km`, `fare_amount`]. SQLSTATE: 42703;

misspelled driver_id to fix I fixed the speliing