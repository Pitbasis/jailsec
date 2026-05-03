# jailsec
jailsec is a tool designed for the FreeBSD operating system that enables the management of Jail environments and the implementation of secure network traffic monitoring. It automates the stages of Jail creation, network interface configuration, and PF firewall installation.  

The script performs the download and extraction of the FreeBSD base system into a user-specified directory. It creates bridge0 and pair interfaces, ensuring stable connectivity between the Host and the Jail. Inside each newly created Jail, a pf.conf file is automatically installed, which includes NAT rules and security filters.  

Beyond its creation functions, jailsec includes a -scan mode. This mode allows for monitoring network activity via tcpdump using BPF filters. The filters are designed to hide the IP addresses of well-known large networks (e.g., Google, Cloudflare) and standard ports (80, 443, 53), making it possible to see only non-standard or suspicious requests.  

To create a new Jail, simply run the script and follow the terminal instructions. During the process, you will need to specify the Jail installation path, preferred IP address, and the interface through which the system connects to the internet. To scan the network, use the -scan flag followed by the interface name or the Jail name.
