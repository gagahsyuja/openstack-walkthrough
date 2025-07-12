<div align=center>
    <img src="assets/openstack-logo.png" width=200 align=center />
</div>
<h1 align=center>OpenStack Walkthrough</h1>

#### A simple guide for getting started with OpenStack Yoga basic deployment on a single node cluster.


# Table of Contents

1. [Learning Resources]()
2. [Requirements]()
2. [Installation Guide]()


## Learning Resources

- [OpenStack](https://www.openstack.org/) - you can pretty much read this and not this guide


## Requirements

- [VirtualBox](https://www.virtualbox.org) - This guide will occurs mostly on a virtual environment, so a hypervisor is needed. In this example, VirtualBox is used.

- [Ubuntu 20.04 LTS](https://releases.ubuntu.com/focal/) - There's a lot of choice regarding what operating system can be used for OpenStack, but because we are using OpenStack Yoga version, we'll be using Ubuntu and 20.04 LTS to be exact.

- [OpenStack Yoga](https://docs.openstack.org/install-guide/) - As we said before, OpenStack Yoga version are what we are going to be install. So it's best to keep their docs opened alongside this guide.

- [Patience](https://en.wikipedia.org/wiki/Patience) - This is going to be a hell of a ride, so enjoy the ride and take a break if needed!

## Installation Guide

At this point, you should have your Ubuntu 20.04 LTS already installed. And with that done, you can proceed to install OpenStack.

#### Network Configuration
Yes, the guide will be starting with configuring the networks, and not installing OpenStack, that will be happened down the road. First of all, there are two kinds of network in OpenStack: 

- **Management Network** - Used for internal communication between OpenStack Components. In this case, because only single node is used, then we can use localhost address (127.0.0.1) for our management network.

- **Provider Network** - This network will provide internet access to the OpenStack environment.

With that said, now we need to configure our hosts configuration at `/etc/hosts` to define 127.0.0.1 as controller. Here's how to do it:

```bash
# /etc/hosts
127.0.0.1	localhost
127.0.1.1	localhost

# OpenStack
127.0.0.1	controller
```

Add `127.0.0.1` and assign `controller` as the hostname. If you pay attention to the hosts entry, there is also another IP that assigned to localhost but not `127.0.0.1`, it's instead `127.0.1.1`. Please comment that out or just get rid of it from the hosts file.

```bash
# /etc/hosts
127.0.0.1	localhost

# OpenStack
127.0.0.1	controller
```

Now, hosts file configuration should be looking similarly like above, containing only `localhost` pointing to `127.0.0.1` and `controller` pointing to `127.0.0.1` as well.

Next, we are going to give our Ubuntu install an internet access. So you should have an IP or interface that provides internet, such as NAT, NAT Networks, or even Bridged Adapter. In this case, Bridged Adapter is what we are going to use, but feel free to choose anything else.

After you decided what is going to be used, now you need to configure the netplan configuration file in order to assign those IP to netplan as our internet configuration. Edit `/etc/netplan/50-cloud-init.yaml` as follows:

```bash
# /etc/netplan/50-cloud-init.yml
network:
  ethernets:
    enp3s0:
      dhcp4: yes
```

If you are using NAT or NAT Networks, DHCP will be available so you don't need to manually define what IP address will be used. But if you need to do it, you can assign IP and gateway like so:

```bash
# /etc/netplan/50-cloud-init.yml
network:
  ethernets:
    enp3s0:
      addresses: [ 192.168.x.x/x ]
      gateway4: 192.168.x.x
```

With that done, now we can verify if we get an internet by pinging an active domain such as `google.com` or anything else:

```bash
$ ping google.com

PING forcesafesearch.google.com (216.239.38.120) 56(84) bytes of data.
64 bytes from any-in-2678.1e100.net (216.239.38.120): icmp_seq=1 ttl=115 time=28.9 ms
64 bytes from any-in-2678.1e100.net (216.239.38.120): icmp_seq=2 ttl=115 time=27.3 ms
64 bytes from any-in-2678.1e100.net (216.239.38.120): icmp_seq=3 ttl=115 time=27.2 ms

--- forcesafesearch.google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 27.154/27.768/28.874/0.783 ms
```

If we get a replies and not timed out like above, you can be sure that you got an internet. Congrats.

Next is to check if our controller IP are correct by pinging it directly:

```bash
ping -c 3 controller

PING controller (::1) 56 data bytes
64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.082 ms
64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.036 ms
64 bytes from localhost (::1): icmp_seq=3 ttl=64 time=0.069 ms

--- controller ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 0.036/0.062/0.082/0.019 ms
```

Again, if we get a replies then everything is good. Congrats.

Congratulations, network configuration is done and you can proceed to the next steps!
