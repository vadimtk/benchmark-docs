.. _aurora-sysbench-2015:

================================
Amazon Aurora Sysbench benchmark
================================

Benchmark date: Nov 2015.

The goal was to evaluate Amazon Aurora performance comparing to Percona Server.
The workload is `sysbench <https://github.com/akopytov/sysbench>`_ v0.5.
Scripts used for testing
* oltp.lua (read-write)
* update_index.lua
* update_non_index.lua

Percona Server version: 5.6.27-75.0

Data sizes
-----------

Initial dataset: * 32 sysbench tables, 50mln rows each. It corresponds to about 400GB of data.
Testing sizes: for benchmarks we vary max amount of rows used by sysbench: 1mln, 2.5mln, 5mln, 10mln, 25mln, 50mln.
In this way we emulate different datasizes from fully in-memory to heavy-IO access.

Instance sizes.
---------------
Actually it is quite complicated to find equal configuration (in both performance and price aspects)
to compare Percona Server running on EC2 instance against Amazon Aurora.

Amazon Aurora:
db.r3.xlarge instance (4 virtual CPUS + 30GB memory)
Monthly computing Cost (1-YEAR TERM, No Upfront): $277.40
Monthly Storage cost: $0.100 per GB-month * 400 GB = $40
+ extra $0.200 per 1 million IO requests

Total cost (excluding extra per IO requests): $311.40


Percona Server
r3.xlarge instance (4 virtual CPUS + 30GB memory)
Monthly computing cost (1-YEAR TERM, No Upfront): $160.6

For the storage we will use 3 options:
* general purpose SSD volume, 500GB size, 1500/3000 ios, cost: $0.10 per GB-month * 500 = $50
* Provisioned IOPS SSD volume, 500GB, 3000 IOP = $0.125 per GB-month  * 500 + $0.065 per provisioned IOPS-month * 3000 = $62.5 + $195 = $257.5
* Provisioned IOPS SSD volume, 500GB, 2000 IOP = $0.125 per GB-month  * 500 + $0.065 per provisioned IOPS-month * 2000 = $62.5 + $130 = $192.5

So corresponding total costs (per month) for used EC2 instances are: $210.6; $418.1; $353.1

Client setup
------------

For client we use t2.medium instance.
Sysbench runs with 16 users threads, which should be adequate to load the database instance.


sysbench script:

.. code-block:: bash

	for i in 50000 25000 10000 5000 2500 1000
	do
	sysbench --test=tests/db/oltp.lua --oltp_tables_count=32 --mysql-user=root --mysql_table_engine=InnoDB --num-threads=16 --oltp-table-size=${i}000 --rand-type=pareto --rand-init=on --report-interval=10 --mysql-host=HOST --mysql-db=sbtest --max-time=7200 --max-requests=0 run | tee -a au.${i}.oltp.txt
	done
ß
