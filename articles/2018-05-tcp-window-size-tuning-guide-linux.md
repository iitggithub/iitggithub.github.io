## TCP Window Size Tuning Guide (Linux)

Tuning TCP window sizes can significantly improve network performance, particularly over long-distance or high-bandwidth connections. This guide provides a practical, step-by-step process for configuring TCP window size parameters on Linux systems. Whether you’re a network administrator or a Linux system engineer, these tips can help optimisae data transmission efficiency.

First of all a few things need to be known prior to tuning. They are:

1. The amount of bandwidth available in Kilobits (Kb)
2. The average ping response time between the source and destination hosts (in milliseconds)
3. The maximum TCP Segment Size (MSS)

For the purpose of this article, let's set some numbers of our own

* Bandwidth (Kb): 20,000 (20Mbps symmetrical link)
* Ping Time to remote server (ms): 340
* Maximum Segment Size: 1300

Note: To make full use of receive or even transmit window size tuning, BOTH hosts should be tuned.

##### First of all, we need to find the current settings for the following parameters:

```
sysctl net.ipv4.tcp_window_scaling        # Enables/disables TCP window scaling (1 = enabled)
sysctl net.ipv4.tcp_slow_start_after_idle # Controls slow start after idle (1 = enabled)
sysctl net.ipv4.tcp_rmem                  # Sets minimum, default, and maximum receive buffer sizes
sysctl net.ipv4.tcp_wmem                  # Sets minimum, default, and maximum send buffer sizes
sysctl net.core.rmem_default              # Sets the default receive buffer size
sysctl net.core.wmem_default              # Sets the default send buffer size
sysctl net.core.rmem_max                  # Maximum receive buffer size allowed
sysctl net.core.wmem_max                  # Maximum send buffer size allowed
sysctl net.core.optmem_max                # Maximum ancillary buffer space
sysctl net.core.netdev_max_backlog        # Maximum packet backlog for each network interface
sysctl net.ipv4.tcp_congestion_control    # Algorithm used for TCP congestion control
sysctl net.ipv4.tcp_timestamps            # Enables/disables TCP timestamps
sysctl net.ipv4.tcp_sack                  # Enables/disables TCP Selective Acknowledgements (SACK)
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

The Bandwidth-Delay Product (BDP) is a critical metric when tuning TCP window sizes. It represents the amount of unacknowledged data that can be in transit to achieve optimal throughput. Calculating BDP helps determine the required buffer size:

1. Convert Bandwidth and Round-Trip Time (RTT):
  * Bandwidth: 20,000 Kbps (20 Mbps)
  * RTT: 340 ms (0.34 seconds)

2. Calculate BDP:

```
BDP = Bandwidth (bits/sec) x RTT (sec)
   = 20,000,000 x 0.34
   = 6,800,000 bits (850,000 bytes)
```

#### Calculate the Optimal TCP Window Size

1. Take the value 65535 which is the maximum unscaled TCP window size and divide 65535 by the Maximum Segment Size (MSS) which in our case is 1300. This determines the number of packets that can fit into a single window.

```
65535 / 1300 = 50.41
```

2. Round down to the nearest even integer to get an even number of segments, resulting in 50.

3. Multiply by MSS: To find the unscaled window size:

```
50 * 1300 = 65000 bytes
```

4. Scale unscaled window size to meet the Bandwidth-Delay Product (BDP)

Multiply 65000 by 2 until the window size exceeds the BDP, calculated previously as 850,000 bytes.

After several multiplications, we get 1,040,000, which is close to our target BDP.

5. Aligning the Window Size with System Constraints

Since 1,040,000 isn’t a directly valid system parameter, divide it by 1024 (to work in 1K blocks) and round up:

```
1,040,000 / 1024 = 1016
```

6. Final Optimal Window Size: Multiply 1016 by 1024 to confirm:

```
1016 * 1024 = 1,040,384 bytes
```

This final value of 1,040,384 bytes is the default window size we’ll use.

7. Setting Minimum and Maximum Window Sizes

#### Determining the Maximum TCP Window Size

The maximum TCP window size is largely determined by the capacity of the network link and the amount of buffer memory available to handle incoming packets. In a high-throughput network, setting an appropriate maximum window size ensures the connection can fully utilise available bandwidth without overwhelming the system.

* Bandwidth-Delay Product (BDP) Reference: The BDP provides a guideline for the maximum window size needed to saturate the link. For instance, if the BDP calculation suggests 16 MB for a 1 Gb/s link, this becomes a reasonable target for the maximum window size.

* Guidelines by Link Capacity:

    * Since the local network link is usually much faster than external links, use the speed of the fastest link for your calculations.
    * For a 1 Gb/s link, a maximum window size of 16 MB (16,777,216 bytes) is typically sufficient.
    * For a 10 Gb/s link, consider increasing the maximum window size to 32 MB or more, as larger buffers can help maintain throughput.

* System Memory Limitations: Ensure that the chosen window size doesn’t exceed the system’s memory allocation limits. Extremely high maximum window sizes can cause excessive memory usage, especially if many connections are active simultaneously.

Example Command: Set the maximum receive window size to 16 MB on a 1 Gb/s network:

```
sysctl -w net.core.rmem_max=16777216
```

#### Determining the Minimum TCP Window Size

Setting the minimum window size helps define the smallest buffer that the system will use for TCP packets, ensuring that even small connections have an adequate starting point. However, setting this too high can lead to excessive memory allocation, particularly if many small connections are active.

* Typical Minimum Window Size: For most general purposes, a minimum window size of 4 KB (4096 bytes) is a good starting point.

* Adjusting Based on Traffic:
    * Low-Latency, High-Frequency Connections (e.g., database or web servers): A lower minimum window (4–8 KB) reduces the risk of overwhelming memory resources.
    * High-Latency or High-Bandwidth Links (e.g., WAN or VPN links): A higher minimum (16–32 KB) can help keep connections performant by reducing the need for TCP slow start during connection setup and will need to be increased from its default (4096) if you set net.ipv4.tcp\_slow\_start\_after\_idle to 0 and disable TCP slow start.
    * Memory Considerations: Setting this value too high may cause issues if the server doesn’t have enough available memory to allocate these buffers across multiple connections.

Example Command: Set the minimum receive window size to 20 KB:

```
sysctl -w net.ipv4.tcp_rmem='20480 1040384 16777216'
```

Here, 20480 is the minimum because we're using a high latecy, low throughput 20Mbps link to the remote server with TCP slow start disabled, 1040384 is the optimal receive window size, and 16777216 is the maximum receive window.

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

##### You can (and definitely should) store these values in a file so they're loaded each time the system boots like so:

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

#### Conclusion and Testing

By tuning TCP window size, you can maximize throughput and minimize latency, particularly in high-latency or high-bandwidth networks. To verify performance improvements:

* Use iperf to measure throughput before and after tuning.
* Use ping tests to observe any reduction in latency.
* If possible, test under realistic network conditions to ensure tuning meets your operational needs.
 
Happy tuning!
