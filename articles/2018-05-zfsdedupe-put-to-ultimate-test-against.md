## OpenZFS/Dedupe Put To The Ultimate Test

### Overview

The aim of this task is to re-evaluate the viability of ZFS in our environment. Concerns were raised over the original test results such as;

1. IO statistics grossly exceed expectations (Could have been influenced by system RAM)
2. Multiple ZFS deadlocks occurring (Possibly due to CPU/RAM contention on the test system)
3. Disk usage savings did not meet or exceed expectations (Possibly due to limiting the amount of user data copied to the machine)

Although the hardware is exactly the same as the previous ZFS test hardware, the results from this round of testing cannot be merged with previous results due to:

1. Changes in drive/raid configuration
2. ZFS record size is set to default (128K)
3. Test server does not have access to the Nimble CS215 SAN

#### Hardware Configuration

The hardware being used for this test is a Dell PowerEdge R720 server with the following hardware configuration:

* 2 x Intel Xeon E5-2667 (3.30GHz, 15MB Cache)
* Intel C600 Chipset
* Memory - 32 GB (4 x 8GB) 1600Mhz DDR3 Registered RDIMMs
* CentOS 7.1 operating system
* PERC H710p Integrated RAID Controller, 1GB NVRAM
* OS storage configuration - 600GB raw storage consisting of 2 x 600GB 15K RPM SAS 6Gbps 2.5in drives in RAID 1
* Data Storage configuration - 1.8TB raw storage consisting of 6 x 600GB 15K RPM SAS 6Gbps 2.5in drives in RAID 10

#### ZFS Benchmark Setup

Because we're using the Dell PERC H710p Integrated RAID Controller, we can't configure ZFS exactly how it prefers to be configured. This would otherwise require JBOD mode, which the H710p card doesn't support. To remedy this, we would need to purchase a PowerVault MD12xx direct attached storage enclosure which includes a PERC H810 RAID Adapter card that does support JBOD mode or (even better) a HBA where ZFS has complete control over the entire world. The more information ZFS has about the disks it's using the better it's able to manage and assess the health of the disks. You can read al about it at [http://open-zfs.org/wiki/Hardware#Hardware_RAID_controllers](http://open-zfs.org/wiki/Hardware#Hardware_RAID_controllers).

We'll also be performing tests with ZFS primarycache set to "all", "metadata" and "none". Just so you are aware, these are the commands you'll need to know in order to change the primarycache setting for the entire pool.

##### Changing the primarycache to "all"

```
$ zfs set primarycache=all zfs
```

##### Create the ZFS pool using /dev/sdb for testing purposes. This will create ~1.62TB storage pool.

```
$ zpool create zfs /dev/sdb
```

##### Create ZFS volume with de-duplication turned on and atime turned off as recommended by some best practices guides.

```
$ zfs create -o de-dup=on -o atime=off zfs/data
```

##### Mount the ZFS volume

```
$ zfs mount zfs/data /data
```

### Test Data

All access to /data is severed to ensure the data contained remained consistent and the tests are not influenced by end users.

There's two types of test data:

1. Compile data
2. User data

#### Compile Data

This is a piece of software to be compiled. To be fair, this could be any piece of software which could be compiled under Linux but the more organisational specific it is, the more relatable it will be to the end users. Since this machine will store software that needs to be compiled anyway, it makes sense to perform a couple of timed compilation tests.

The test data is stored under /data (wherever is convenient) and will be accessed from a remote "helper" server via NFS. More on that later..

#### User Data

This is the reason why we're here. How much of this data can fit into 1.8TB of raw storage. Our users data (known herein as user data) will be migrated via rsync to the test server. How many exactly? enough to fill 100% of the space available. That should be quite a bit... we might not have enough data.

### Testing Process

A total of 3 tests will be performed. All of which are outlined below;

1. Compilation Speed
2. Disk Space Consolidation
3. IO Tests

#### Compilation Speed

This test aims to compare ZFS using an organisational specific metric that everyone is able to understand - software compilation times. The compilation test is executed three times, timed and the resulting run times are then averaged. Flock is used to make sure that compilation times are not influenced by cache.

This test requires the use of a "helper" server. This server should be be unused and ideally connected to the same network switch as the test server. This is to ensure that the testing environment remains consistent throughout the test.

Also since we'll be accessing the data via NFS, it's important set the mount options exactly as they are in production. Failure to do so could impact test results.

##### Mount the test directory via NFS

```
$ sudo mkdir -p /mnt/compile_test && sudo mount -t nfs -o
"vers=3,rsize=8192,wsize=8192,soft" zfs:/data/compile_test
/mnt/compile_test
```

### Testing Procedure

From within the testing directory, execute the following:

```
$ flock /tmp/compile.lock -c "make veryclean && time make -j `grep processor -c /proc/cpuinfo`"
```

### Results

The results show that there is very little performance difference when using ZFS with in-line deduplication. This is very interesting because the previous compile tests showed a 25% difference which i now believe was due to the fact that we were attempting a CPU intensive workload on the same machine which was running ZFS. For me, it's validation that says we should never use a ZFS server for anything other than ZFS unless it's absolutely necessary.

|  | Run 1 | Run 2 | Run 3 | Average | % Difference |
|---|---|---|---|---|---|
| EXT4 Logical Volume | 366 | 380 | 371 | 372 | +00.00% |
| ZFS Primary Cache: all | 388 | 364 | 370 | 374 | +00.54% |
| ZFS Primary Cache: metadata | 520 | 599 | 568 | 562 | +51.08% |
| ZFS Primary Cache: none | 626 | 654 | 634 | 638 | +71.51% |