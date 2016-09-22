HDFS Lab: Test HDFS performance

Create a ~10 GB file using teragen
Set the block size to 32 MB for this file
Use the time command to record the job duration
2m47.757s
time hadoop jar hadoop-examples.jar teragen -D dfs.block.size=33554432 107374182 10G

Run the terasort command against this file twice
Use the time command to capture each run's duration
Create a cache pool and directive for the teragen output before the first run
Record your work, including:
The teragen command you used to create your test file
The commands you used to configure HDFS caching
The time output for each job run
Store and comment on these outputs in storage/labs/1_terasort_tests.md

hdfs cacheadmin -addDirective -path /user/bob/10G -pool dawgpool
hdfs cacheadmin -addPool dawgpool -owner bob

time hadoop jar hadoop-examples.jar terasort 10G 10G-sort1
16/09/21 17:22:19 INFO terasort.TeraSort: done

real    7m42.409s
user    0m8.867s
sys     0m0.362s

16/09/21 17:36:41 INFO terasort.TeraSort: done

real    7m23.238s
user    0m9.804s
sys     0m0.378s

time hadoop jar hadoop-examples.jar terasort 10G 10G-sort2

time hadoop jar hadoop-examples.jar terasort 10G 10G-sort2
