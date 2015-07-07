.. _mongodb-sysbench-hppro2:

==================
MongoDB sysbench-mongo benchmark
==================

Benchmark date: Jun 2015.

The goal was to evaluate different available storage engines for MongoDB.
The workload is `sysbench-mongodb <https://github.com/tmcallaghan/sysbench-mongodb>`_.
The load is designed to be a heavy IO-load, and I use a slow storage.

Workload description
====================
* NUM_COLLECTIONS=640
* NUM_DOCUMENTS_PER_COLLECTION=1000000
* NUM_WRITER_THREADS=64
* RUN_TIME_MINUTES=180 (in some cases 60)

This gives about ~250GB of data for MMAP engines (uncompressed).
I use small cache sizes and limit total available memory available for mongod process via cgroups
`see cgroup blog post <https://www.percona.com/blog/2015/07/01/using-cgroups-to-limit-mysql-and-mongodb-memory-usage/>`_

So I use following cache/total memory configurations:

* 8GB cache / 16GB total
* 16GB cache / 32GB total
* 32GB cache / 48GB total

For MMAP engine there is no cache configuration, so only total memory limit is applied.

I use prolonged runs (1h or 3h) with measurements every 10sec to see long-term patterns and trends.

I use my binaries for MongoDB which includes RocksDB storage engine `link to binaries <http://percona-lab-mongorocks.s3.amazonaws.com/mongo-rocks-3.0.4-pre-STATIC.tar.gz>`_

MMAP vs WiredTiger
==================

MongoDB MMAP startup command line

	``$MONGODIR/mongod --dbpath=$DATADIR --logpath=$1/server.log``

MongoDB wiredTiger startup command line

	``$MONGODIR/mongod --dbpath=$DATADIR --storageEngine=wiredTiger --wiredTigerCacheSizeGB=X --wiredTigerJournalCompressor=none --logpath=$1/server.log``

Results:
--------

The graphical results to see the pattern (throughput is shown, in operations per second, more is better)

.. image:: img/wt-mmap.png

Or average results (throughput, operations per second) during 2nd hour:

=====  ==== ==========
size   mmap wiredTiger
=====  ==== ==========
8/16   26   53
16/32  31   61
32/48  35   67
=====  ==== ==========

RocksDB vs WiredTiger
==================

MongoDB RocksDB startup command line

	``$MONGODIR/mongod --dbpath=$DATADIR --storageEngine=rocksdb --rocksdbCacheSizeGB=X``

Most runs for RocksDB was 60 min, plus one extra run for 180 min.

.. image:: img/wt-rocksdb.png

=====  ==== ========== ========
size   mmap wiredTiger RocksDB
=====  ==== ========== ========
8/16   26   53         81
16/32  31   61         91
32/48  35   67         104 
=====  ==== ========== ========



