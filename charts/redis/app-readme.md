# Redis

## Topology

````txt
                   +----+
                   | M1 |
                   | S1 |
                   +----+
                      |
+----+       +----+   |   +----+       +----+
| R2 |---+---| R3 |---+---| R4 |---+---| R5 |
| S2 |       | S3 |       | S4 |       | S5 |
+----+       +----+       +----+       +----+
````

If the master M1 fails, S2 and S3 will agree about the failure and will be able to authorize a failover, making clients able to continue.

In every Sentinel setup, being Redis asynchronously replicated, there is always the risk of losing some write because a given acknowledged write may not be able to reach the slave which is promoted to master. However in the above setup there is an higher risk due to clients partitioned away with an old master.

In this case a network partition isolated the old master M1, so the slave R2 is promoted to master. However clients, like C1, that are in the same partition as the old master, may continue to write data to the old master. This data will be lost forever since when the partition will heal, the master will be reconfigured as a slave of the new master, discarding its data set.

This problem can be mitigated using the following Redis replication feature, that allows to stop accepting writes if a master detects that is no longer able to transfer its writes to the specified number of slaves.

````txt
min-slaves-to-write 1
min-slaves-max-lag 10
````

With the above configuration (please see the self-commented redis.conf example in the Redis distribution for more information) a Redis instance, when acting as a master, will stop accepting writes if it can't write to at least 1 slave. Since replication is asynchronous not being able to write actually means that the slave is either disconnected, or is not sending us asynchronous acknowledges for more than the specified max-lag number of seconds.

More on:
- [redis.conf](http://download.redis.io/redis-stable/redis.conf);
- [redis sentinel documentation](https://redis.io/topics/sentinel)
