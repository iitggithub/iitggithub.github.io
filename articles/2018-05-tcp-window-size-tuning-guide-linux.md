## TCP Window Size Tuning Guide (Linux)

There are many ways to tune TCP window sizes, below i show you how i do it. Feel free to comment because i'm not 100% sure on this but i'd like to document this process as I remember it anyway.

First of all a few things need to be known prior to tuning. They are:

1. The amount of bandwidth available in Kb
2. The average ping response time between the source and destination hosts (in ms)
3. The maximum TCP segment size

For the purpose of this article, let's set some numbers of our own

* Bandwidth (Kb): 20,000
* Ping Time (ms): 340
* Max Segment Size: 1300

Note: To make full use of receive or even transmit window size tuning, BOTH hosts should be tuned.

##### First of all, we need to find the current settings for the following parameters:

```
sysctl net.ipv4.tcp_window_scaling
sysctl net.ipv4.tcp_slow_start_after_idle
sysctl net.ipv4.tcp_rmem
sysctl net.ipv4.tcp_wmem
sysctl net.core.rmem_default
sysctl net.core.wmem_default
sysctl net.core.rmem_max
sysctl net.core.wmem_max
sysctl net.core.optmem_max
sysctl net.core.netdev_max_backlog
sysctl net.ipv4.tcp_congestion_control
sysctl net.ipv4.tcp_timestamps
sysctl net.ipv4.tcp_sack
```

##### As an example, this is the output when running those commands on our test system: 

```
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_slow_start_after_idle = 1
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
net.core.rmem_default = 124928
net.core.wmem_default = 124928
net.core.rmem_max = 124928
net.core.wmem_max = 124928
net.core.optmem_max = 20480
net.core.netdev_max_backlog = 1000
net.ipv4.tcp_congestion_control = cubic
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_sack = 1
```

We’re only going to do receive window tuning so ignore the wmem parameters (it’s the same process anyway but requires bi-directional access in order to test).

#### Calculating The Optimal Window Size

First we need to find our BDP or Bandwidth Delay Product.

Start by multiplying the amount of bandwidth available (in our case, is 20000) by the ping response time (340ms). This gives us 6800000. Divide this by 8 and we have our BDP which is 850000.

Now we need to find our unscaled window value.

Take 65535 (why 65535? I don’t know) and divide it by our MSS (1300) and round down the result to the nearest even number. 65535 / 1300 is 50.41153846153846. Rounded down to the nearest even number brings us to 50. Then, multiply this value by 1300 (our MSS) to find the optimal unscaled window value which is 65000.

Still following? Hope so.

Multiply 65000 by 2 until it is larger than our BDP (which is 850000) and you should arrive at 1040000 which is your optimal window size. You can’t use 1040000 because it’s not a valid number. I usually divide it by 1024 and round up the result to the nearest whole number. In our case, 104000 / 1024 is 1015.625 which when rounded up is 1016. 1024 * 1016 = 1040384 which after all that is now our default window size. That’s just the method I use but you can do more research if you want to but I find it easy to remember especially since 1024 is usually the default minimum value given when you dump the current configuration.

We also need to set a minimum window size and a maximum window size. To save you some time and effort, use 16MB or 16777216 which is the maximum for a 1Gb/s local network link. Of course, if the server has a larger local network link ie 10Gb/s the maximum should be at least double that.

Minimum receive window size can be set where ever you want. Set this value too high and It will cause problems because there isn’t enough memory available to handle your minimum.


#### The Optimised Values

Note that this also includes write optimisation (wmem) values as well because i find myself referencing this documentation a lot.

```
# Turn on automatic TCP window size scaling
sysctl -w net.ipv4.tcp_window_scaling=1
# Disable TCP slow start
sysctl -w net.ipv4.tcp_slow_start_after_idle=0
# Set the min, default and maximum receive window sizes used during auto tuning
sysctl -w net.ipv4.tcp_rmem='20480 1040384 16777216'
sysctl -w net.ipv4.tcp_wmem='20480 1040384 16777216'
# Set default receive window size here as well. This one is used when window size scaling is disabled
sysctl -w net.core.rmem_default=1040384
sysctl -w net.core.wmem_default=1040384
# Set max receive window size here as well. This one is used when window size scaling is disabled
sysctl -w net.core.rmem_max=16777216
sysctl -w net.core.wmem_max=16777216
# Set the maximum buffer size allowed per socket
# I recommend just setting this to your maximum window size
sysctl -w net.core.optmem_max=16777216
# Sets the maximum number of packets that will be buffered if the kernel can’t keep up
# There’s no real method, I just set it to something that’s a lot higher than default
sysctl -w net.core.netdev_max_backlog=65536
# Set the congestion control algorithm. Not sure which one is better but cubic seems like it’s better than reno
sysctl -w net.ipv4.tcp_available_congestion_control=’cubic’
# Enable timestamps as defined in RFC1323
sysctl -w net.ipv4.tcp_timestamps=1
# Enable select acknowledgments
sysctl -w net.ipv4.tcp_sack=1
# Force all new TCP connections to use the above settings
sysctl -w net.ipv4.route.flush=1
```

##### You can (and definitely should) store these values in a file like so:

```
$ cat | tee /etc/sysctl.d/tcp_optimisations.conf <<EOF
# Turn on automatic TCP window size scaling
net.ipv4.tcp_window_scaling = 1
# Disable TCP slow start
net.ipv4.tcp_slow_start_after_idle = 0
# Set the min, default and maximum receive window sizes used during auto
tuning
net.ipv4.tcp_rmem = 20480 1040384 16777216
net.ipv4.tcp_wmem = 20480 1040384 16777216
# Set default receive window size here as well. This one is used when
window size scaling is disabled
net.core.rmem_default = 1040384
net.core.wmem_default = 1040384
# Set max receive window size here as well. This one is used when window
size scaling is disabled
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
# Set the maximum buffer size allowed per socket
# I recommend just setting this to your maximum window size
net.core.optmem_max = 16777216
# Sets the maximum number of packets that will be buffered if the kernel
can’t keep up
# There’s no real method, I just set it to something that’s a lot higher
than default
net.core.netdev_max_backlog = 65536
# Set the congestion control algorithm. Not sure which one is better but
cubic seems like it’s better than reno
net.ipv4.tcp_available_congestion_control = cubic
# Enable timestamps as defined in RFC1323
net.ipv4.tcp_timestamps = 1
# Enable select acknowledgments
net.ipv4.tcp_sack = 1
# Force all new TCP connections to use the above settings
net.ipv4.route.flush = 1
EOF
```

 ... and that's it! Happy tuning!
