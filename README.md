# Optimizing System Performance for SIP servers

There are many guides online about Linux kernel and TCP/UDP tuning, I tried to sum the most useful and detailed Linux kernel useful to scale and handle more concurrent connections on a SIP server such as Asterisk or Kamailio.

This is the **/etc/sysctl.conf** file I use on my servers (CentOS 6):

```
#
# Tunned Kernel sysctl configuration file for Linux
#
# Version 1.0 - 2018-02-15
# Rebtel - <oussama.hammami@rebtel.com>
#
# This file should be saved in /etc/sysctl.d/ and can be activated using the command:
# sysctl -e -p /etc/sysctl.d/70-rebtel-tuning.conf
#

# Disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 0

# Usually SIP uses TCP or UDP to carry the SIP signaling messages over the internet (<=> TCP/UDP sockets).
# The receive buffer (socket receive buffer) holds the received data until it is read by the application.
# The send buffer (socket transmit buffer) holds the data until it is read by the underling protocol in the network stack.

#net.core.rmem_max = 10485760
#net.core.rmem_max = 12582912
#net.core.rmem_max = 33554432
net.core.rmem_max = 67108864

#net.core.wmem_max = 10485760
#net.core.wmem_max = 12582912
net.core.wmem_max = 33554432

#net.core.rmem_default = 10485760
net.core.rmem_default = 31457280
#net.core.wmem_default = 10485760
net.core.wmem_default = 31457280

net.ipv4.tcp_rmem = 10240 87380 10485760
net.ipv4.tcp_wmem= 10240 87380 10485760

# Increase the write-buffer-space allocatable
net.ipv4.udp_rmem_min = 131072
net.ipv4.udp_wmem_min = 131072
# net.ipv4.udp_mem = 65536 131072 262144
net.ipv4.udp_mem = 19257652 19257652 19257652
net.ipv4.tcp_mem = 786432 1048576 26777216

# Increase the maximum amount of option memory buffers
net.core.optmem_max = 25165824

# allow services to bind to the virtual ip even when this server is the passive machine
net.ipv4.ip_nonlocal_bind = 1

# Disable TCP timestamp (RFC 1321): TCP timestamp feature allows round trip time measurement (<=>  Adding 8 bytes to TCP header).
# To avoid this overhead we disable this feature:
net.ipv4.tcp_timestamps = 0

# Enable window scaling:
net.ipv4.tcp_window_scaling = 1

# Disable select acknowledgements (SACK):
net.ipv4.tcp_sack = 1

# Disable cache metrics so the initial conditions of the closed connections will not be saved to be used in near future connections:
net.ipv4.tcp_no_metrics_save = 1

# Tune the value of "backlog" (maximum queue length of pending connections "Waiting Acknowledgment"):
net.ipv4.tcp_max_syn_backlog = 300000

# Set the value of somaxconn. This is the Max value of the backlog. The default value is 128.
# If the backlog is greater than somaxconn, it will truncated to it.
net.core.somaxconn = 65535

# The kernel parameter "netdev_max_backlog" is the maximum size of the receive queue.
net.core.netdev_max_backlog = 300000

# TIME_WAIT TCP socket state is the state where the socket is closed but waiting to handle the packets which are still in the network.
# The parameter tcp_max_tw_buckets is the maximum number of sockets in TIME_WAIT state.
# After reaching this number the system will start destroying the socket in this state.
# Increase the tcp-time-wait buckets pool size to prevent simple DOS attacks
net.ipv4.tcp_max_tw_buckets = 1440000
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_tw_reuse = 1

# How often the keepalive packets will be sent to keep the connection alive
# Set the keep-alives to less than 600 seconds to ensure that connections
# are refreshed before the GCP timeout occurs
net.ipv4.tcp_keepalive_time = 60
net.ipv4.tcp_keepalive_probes = 5
# time to wait for a reply on each keepalive probe
net.ipv4.tcp_keepalive_intvl = 60

# how many times to retry before killing an alive TCP connection
net.ipv4.tcp_retries2 = 5

# how many times to retransmit the initial SYN packet
net.ipv4.tcp_syn_retries = 5

# Decrease the time default value for tcp_fin_timeout connection
net.ipv4.tcp_fin_timeout = 15

# check if the forwarding is necessary for rtprngine and rtpproxy
#net.ipv4.ip_forward=1

# change the maximum number of open files
# be sure that /proc/sys/fs/inode-max is 3-4 times the new value of
# /proc/sys/fs/file-max, or you will run out of inodes.
# The upper limit on fs.file-max is recorded in fs.nr_open (which is 1024*1024)
fs.file-max = 500000

# The value 0 makes the kernel swap only to avoid out of memory condition.
# Do less swapping
vm.swappiness = 10
vm.dirty_ratio = 60
vm.dirty_background_ratio = 2

# The default operating system limits on mmap counts is likely to be too low
# used by vmtouch
vm.max_map_count=262144

# the maximum size (in bytes) of a single shared segment that a Linux process can allocate in its virtual address space.
# 1/2 of physical RAM,  shared memory segment theoretically is 2^64bytes. This is correspond to all physical RAM that you have.
kernel.shmmax = 1073741824

# total port range
net.ipv4.ip_local_port_range = 1024 65535


# Provide protection from ToCToU races
fs.protected_hardlinks=1

# Provide protection from ToCToU races
fs.protected_symlinks=1

# Make locating kernel addresses more difficult
kernel.kptr_restrict=1

# Set ptrace protections
kernel.yama.ptrace_scope=1

# Set perf only available to root
kernel.perf_event_paranoid=2

# Turn on SYN-flood protections.  Starting with 2.6.26, there is no loss
# of TCP functionality/features under normal conditions.  When flood
# protections kick in under high unanswered-SYN load, the system
# should remain more stable, with a trade off of some loss of TCP
# functionality/features (e.g. TCP Window scaling).
net.ipv4.tcp_syncookies=1

# Ignore source-routed packets
net.ipv4.conf.all.accept_source_route=0
net.ipv4.conf.default.accept_source_route=0

# Ignore ICMP redirects from non-GW hosts
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.default.accept_redirects=0
net.ipv4.conf.all.secure_redirects=1
net.ipv4.conf.default.secure_redirects=1

# Don't pass traffic between networks or act as a router
net.ipv4.ip_forward=0
net.ipv4.conf.all.send_redirects=0
net.ipv4.conf.default.send_redirects=0

# Turn on Source Address Verification in all interfaces to
# prevent some spoofing attacks.
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.default.rp_filter=1

# Ignore ICMP broadcasts to avoid participating in Smurf attacks
net.ipv4.icmp_echo_ignore_broadcasts=1

# Ignore bad ICMP errors
net.ipv4.icmp_ignore_bogus_error_responses=1

# Log spoofed, source-routed, and redirect packets
net.ipv4.conf.all.log_martians=1
net.ipv4.conf.default.log_martians=1

# RFC 1337 fix
net.ipv4.tcp_rfc1337=1

# Addresses of mmap base, heap, stack and VDSO page are randomized
kernel.randomize_va_space=2

# Reboot the machine soon after a kernel panic.
kernel.panic=10
```

And this is the **/etc/security/limits.conf** :

```
#
# Limits configuration for ulimit.
#
# Version 1.0 - 2018-02-21
# Rebtel - <oussama.hammami@rebtel.com>
#
# This file should be saved in /etc/security/limits.d and can be activated by closing all
# active sessions of the concerned user:
# ulimit -a
#
# maximum number of open files
# don't use value upper than fs.file-max
root       soft    nofile         102400
root       hard    nofile         102400
# limits the core file size (KB)
root       soft    core            unlimited
# maximum data size (KB)
root       soft    data            unlimited
# maximum filesize (KB)
root       soft    fsize           unlimited
# maximum locked-in-memory address space (KB)
root       soft    memlock         unlimited
root       hard    memlock         unlimited
# maximum resident set size (KB) (Ignored in Linux 2.4.30 and higher)
# root       soft    rss             unlimited
# maximum stack size (KB)
root       soft    stack           unlimited
root       hard    stack           unlimited
# maximum CPU time (minutes)
root       soft    cpu             unlimited
# maximum number of processes
root       soft    nproc           unlimited
root       hard    nproc           unlimited
# address space limit (KB)
root       soft    as              unlimited
# the priority to run user process with (negative values boost process priority)
root       soft    priority        -11
# maximum locked files (Linux 2.4 and higher)
root       soft    locks           unlimited
# maximum number of pending signals (Linux 2.6 and higher)
root       soft    sigpending      unlimited
root       hard    sigpending      unlimited
# maximum memory used by POSIX message queues (bytes) (Linux 2.6 and higher)
root       soft    msgqueue        unlimited
root       hard    msgqueue        unlimited
# maximum nice priority allowed to raise to (Linux 2.6.12 and higher) values: [-20,19]
root       soft    nice            -11
```

To add a new user limits, you can copy paste this file in `/etc/security/limits.d/` and replace root by your process name.

**Example** for asterisk:

```
# sed -i 's/root/asterisk/g' 50-asterisk.limits.conf
``` 

# Increasing the Limit in Systemd

> |Note that systemd ignores limits set in the /etc/security/limits.conf and /etc/security/limits.d/*.conf configuration files. The limits defined in these files are set by PAM when starting a login session, but daemons started by systemd do not use PAM login sessions.

### Asterisk service `/usr/lib/systemd/system/asterisk.service`

```
[Unit]
Description=Asterisk PBX and telephony daemon.
After=network.target

[Service]
Type=simple
Environment=HOME=/var/lib/asterisk
WorkingDirectory=/var/lib/asterisk
User=asterisk
Group=asterisk
ExecStart=/usr/sbin/asterisk -f -C /etc/asterisk/asterisk.conf
ExecStop=/usr/sbin/asterisk -rx 'core stop now'
ExecReload=/usr/sbin/asterisk -rx 'core reload'

# To emulate some of the features of the safe_asterisk script, copy
# this file to /etc/systemd/system/asterisk.service and uncomment one
# or more of the following lines.  For more information on what these
# parameters mean see:
#
# http://0pointer.de/public/systemd-man/systemd.service.html
# http://0pointer.de/public/systemd-man/systemd.exec.html

Nice=1
#UMask=0002
LimitCORE=infinity
LimitNPROC=infinity
LimitAS=infinity
LimitRSS=infinity
LimitDATA=infinity
LimitFSIZE=infinity
TimeoutSec=300
LimitNOFILE=1024000
LimitSTACK=infinity
TasksMax=infinity
LimitRTPRIO=70
MemoryLimit=infinity
LimitSIGPENDING=infinity
LimitMSGQUEUE=infinity
LimitMEMLOCK=infinity

#Restart=always
#RestartSec=4

# If you uncomment the following you should add '-c' to the ExecStart line above

#TTYPath=/dev/tty7
#StandardInput=tty
#StandardOutput=tty
#StandardError=tty

PrivateTmp=true

[Install]
WantedBy=multi-user.target
*Note that systemd ignores limits set in the /etc/security/limits.conf and /etc/security/limits.d/*.conf configuration files. The limits defined in these files are set by PAM when starting a login session, but daemons started by systemd do not use PAM login sessions.*
```
to check the limits you can use:

```
# cat /proc/`pidof asterisk`/limits
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            unlimited            unlimited            bytes
Max core file size        unlimited            unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             unlimited            unlimited            processes
Max open files            1024000              1024000              files
Max locked memory         unlimited            unlimited            bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       unlimited            unlimited            signals
Max msgqueue size         unlimited            unlimited            bytes
Max nice priority         0                    0
Max realtime priority     70                   70
Max realtime timeout      unlimited            unlimited            us
```

## Reference

* [Linux Tuning For SIP Routers](https://voipmagazine.wordpress.com/2014/12/13/linux-tuning-for-sip-routers-part-1-interrupts-and-irq-tuning/)

* [Google Cloud Kernel security settings](https://cloud.google.com/compute/docs/images/building-custom-os#kernelsecurity)

* [Sysctl tuning for optimized system performance](https://mindless.atlassian.net/wiki/spaces/Linux/pages/1114116/Sysctl+tuning+for+optimized+system+performance)

* [Securing Network Access](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-securing_network_access)
