# Best practices for configuring network interfaces<a name="best-practices-for-configuring-network-interfaces"></a>
+ You can attach a network interface to an instance when it's running \(hot attach\), when it's stopped \(warm attach\), or when the instance is being launched \(cold attach\)\.
+ You can detach secondary network interfaces when the instance is running or stopped\. However, you can't detach the primary network interface\.
+ You can move a network interface from one instance to another, if the instances are in the same Availability Zone and VPC but in different subnets\.
+ When launching an instance using the CLI, API, or an SDK, you can specify the primary network interface and additional network interfaces\.
+ Launching an Amazon Linux or Windows Server instance with multiple network interfaces automatically configures interfaces, private IPv4 addresses, and route tables on the operating system of the instance\.
+ A warm or hot attach of an additional network interface might require you to manually bring up the second interface, configure the private IPv4 address, and modify the route table accordingly\. Instances running Amazon Linux or Windows Server automatically recognize the warm or hot attach and configure themselves\.
+ You cannot attach another network interface to an instance \(for example, a NIC teaming configuration\) to increase or double the network bandwidth to or from the dual\-homed instance\.
+ If you attach two or more network interfaces from the same subnet to an instance, you might encounter networking issues such as asymmetric routing\. If possible, use a secondary private IPv4 address on the primary network interface instead\.

## Configure your network interface using ec2\-net\-utils for Amazon Linux<a name="ec2-net-utils"></a>

Amazon Linux AMIs may contain additional scripts installed by AWS, known as ec2\-net\-utils\. These scripts optionally automate the configuration of your network interfaces\. These scripts are available for Amazon Linux only\.

Use the following command to install the package on Amazon Linux if it's not already installed, or update it if it's installed and additional updates are available:

```
$ yum install ec2-net-utils
```

The following components are part of ec2\-net\-utils:

udev rules \(`/etc/udev/rules.d`\)  
Identifies network interfaces when they are attached, detached, or reattached to a running instance, and ensures that the hotplug script runs \(`53-ec2-network-interfaces.rules`\)\. Maps the MAC address to a device name \(`75-persistent-net-generator.rules`, which generates `70-persistent-net.rules`\)\.

hotplug script  
Generates an interface configuration file suitable for use with DHCP \(`/etc/sysconfig/network-scripts/ifcfg-eth`*N*\)\. Also generates a route configuration file \(`/etc/sysconfig/network-scripts/route-eth`*N*\)\.

DHCP script  
Whenever the network interface receives a new DHCP lease, this script queries the instance metadata for Elastic IP addresses\. For each Elastic IP address, it adds a rule to the routing policy database to ensure that outbound traffic from that address uses the correct network interface\. It also adds each private IP address to the network interface as a secondary address\.

ec2ifup eth*N*  
Extends the functionality of the standard ifup\. After this script rewrites the configuration files `ifcfg-eth`*N* and `route-eth`*N*, it runs ifup\.

ec2ifdown eth*N*  
Extends the functionality of the standard ifdown\. After this script removes any rules for the network interface from the routing policy database, it runs ifdown\.

ec2ifscan  
Checks for network interfaces that have not been configured and configures them\.  
This script isn't available in the initial release of ec2\-net\-utils\.

To list any configuration files that were generated by ec2\-net\-utils, use the following command:

```
$ ls -l /etc/sysconfig/network-scripts/*-eth?
```

To disable the automation, you can add `EC2SYNC=no` to the corresponding `ifcfg-eth`*N* file\. For example, use the following command to disable the automation for the eth1 interface:

```
$ sed -i -e 's/^EC2SYNC=yes/EC2SYNC=no/' /etc/sysconfig/network-scripts/ifcfg-eth1
```

To disable the automation completely, you can remove the package using the following command:

```
$ yum remove ec2-net-utils
```