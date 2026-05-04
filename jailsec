#!/bin/sh

if [ "$1" = "-scan" ]; then
  if [ -z "$2" ]; then
  echo "|--------------------------------|"
  echo "|                                |"
  echo "| -scan [wlan0/re0] [jail_name]  |"
  echo "|                                |"
  echo "|_______________or_______________|"
  echo "|                                |" 
  echo "| -scan [wlan0/re0]              |"
  echo "|                                |"
  echo "|________________________________|"
  fi
  if [ -n  "$2" ] && [ -z "$3" ]; then
     wifi="$2"
     IGNORE_NETS="
     142.250.0.0/15
     142.251.0.0/16
     172.217.0.0/16
     192.178.0.0/15
     216.58.192.0/19
     172.64.0.0/13
     104.16.0.0/12
     151.101.0.0/16
     212.73.87.0/24
     178.160.192.0/18
     34.107.0.0/16
     "

     IGNORE_IPS=$(sudo sockstat -46c | awk 'NR>1 {print $7}' | rev | cut -d: -f2- | rev | grep -v '^\*$' | sort -u)

     BPF_FILTER=""

     for NET in $IGNORE_NETS; do
        if [ -z "$BPF_FILTER" ]; then
            BPF_FILTER="not net $NET"
        else
            BPF_FILTER="$BPF_FILTER and not net $NET"
        fi
     done

     for IP in $IGNORE_IPS; do
        if [ -z "$BPF_FILTER" ]; then
           BPF_FILTER="not host $IP"
        else
           BPF_FILTER="$BPF_FILTER and not host $IP"
        fi
    done

    if [ -n "$BPF_FILTER" ]; then
       BPF_FILTER="$BPF_FILTER and not arp and not ether proto 0x893a and not port 53 and not ip6 and not ether proto 0x88cc and not net 224.0.0.0/4 and not net 239.0.0.0/8 and not igmp and not port 80 and not port 443"
    else
       BPF_FILTER="not arp and not ether proto 0x893a and not port 53 and not ip6 and not ether proto 0x88cc and not net 224.0.0.0/4 and not net 239.0.0.0/8 and not igmp and not port 80 and not port 443"
    fi
    echo "[*] network filter [*] $wifi "
    sudo tcpdump -ni $wifi $BPF_FILTER
  elif [ -n "$3" ]; then 
    wifi="$2"
    jail="$3"
   IGNORE_NETS="
   142.250.0.0/15
   142.251.0.0/16
   172.217.0.0/16
   192.178.0.0/15
   216.58.192.0/19
   172.64.0.0/13
   104.16.0.0/12
   151.101.0.0/16
   212.73.87.0/24
   178.160.192.0/18
   34.107.0.0/16
   "

   IGNORE_IPS=$(sudo sockstat -46c | awk 'NR>1 {print $7}' | rev | cut -d: -f2- | rev | grep -v '^\*$' | sort -u)

   BPF_FILTER=""

   for NET in $IGNORE_NETS; do
     if [ -z "$BPF_FILTER" ]; then
        BPF_FILTER="not net $NET"
     else
        BPF_FILTER="$BPF_FILTER and not net $NET"
     fi
   done

   for IP in $IGNORE_IPS; do
    if [ -z "$BPF_FILTER" ]; then
        BPF_FILTER="not host $IP"
    else
        BPF_FILTER="$BPF_FILTER and not host $IP"
    fi
   done
   if [ -n "$BPF_FILTER" ]; then
    BPF_FILTER="$BPF_FILTER and not arp and not ether proto 0x893a and not port 53 and not ip6 and not ether proto 0x88cc and not net 224.0.0.0/4 and not net 239.0.0.0/8 and not igmp and not port 80 and not port 443"
   else
    BPF_FILTER="not arp and not ether proto 0x893a and not port 53 and not ip6 and not ether proto 0x88cc and not net 224.0.0.0/4 and not net 239.0.0.0/8 and not igmp and not port 80 and not port 443"
   fi
   echo "[*] network filter [$2] jail -> $3 "
   sudo jexec $jail tcpdump -ni $wifi $BPF_FILTER
  fi 
else 
FETCH_DEST="/tmp/base.txz"
CONF_FILE="/etc/jail.conf"
GATEWAY_IP="10.0.0.2"

printf "Enter the full path for the jail (e.g., /tmp/jails/j1_pf): "
read JAIL_PATH
[ -z "$JAIL_PATH" ] && exit 1
JAIL_NAME=$(basename "$JAIL_PATH")

echo "-----------------------------------"
echo "Select your Internet Interface (e.g. re0 or wlan0):"
i=1
INTERFACES=$(ifconfig -l)
for iface in $INTERFACES; do
    echo "$i. $iface"
    eval "iface_$i=$iface"
    i=$((i + 1))
done
printf "Select number: "
read IFACE_NUM
EXT_IF=$(eval echo \$iface_$IFACE_NUM)
echo "$EXT_IF"
printf "Enter Jail IP (e.g., 10.0.0.1): "
read JAIL_IP

mkdir -p "$JAIL_PATH"

if [ ! -s "$FETCH_DEST" ]; then
    VERSION=$(freebsd-version | cut -d'-' -f1,2)
    ARCH=$(uname -m)
    echo "Downloading FreeBSD $VERSION base..."
    fetch -o "$FETCH_DEST" "https://download.freebsd.org/ftp/releases/$ARCH/$VERSION/base.txz"
fi

echo "Extracting system to $JAIL_PATH..."
tar -xf "$FETCH_DEST" -C "$JAIL_PATH"

echo "Configuring Network Interfaces..."

ifconfig bridge0 create >/dev/null 2>&1
ifconfig epair0 create >/dev/null 2>&1

ifconfig bridge0 up
ifconfig epair0b inet $GATEWAY_IP netmask 255.255.255.0 up
ifconfig bridge0 addm epair0b addm epair0a inet 10.0.0.2/24 up

sysrc cloned_interfaces+="bridge0 epair0"
sysrc ifconfig_bridge0=" addm epair0b addm epair0a inet 10.0.0.2/24 up"
sysrc ifconfig_epair0b="up"
sysrc ifconfig_epair0a="inet 10.0.0.2 netmask 255.255.255.0 up"

echo "Updating $CONF_FILE..."
cat <<EOF >> $CONF_FILE

$JAIL_NAME {
    host.hostname = "$JAIL_NAME";
    path = "$JAIL_PATH";
    vnet;
    vnet.interface = "epair0b", "$EXT_IF";
    mount.devfs;
    devfs_ruleset = "0";
    exec.start = "/bin/sh /etc/rc";
    exec.stop = "/bin/sh /etc/rc.shutdown";
    exec.clean;
    allow.raw_sockets;
}
EOF

echo "Setting up networking inside $JAIL_NAME/etc/rc.conf..."
JAIL_RC="$JAIL_PATH/etc/rc.conf"
touch "$JAIL_RC"

sysrc -f "$JAIL_RC" ifconfig_epair0b="inet $JAIL_IP netmask 255.255.255.0"
sysrc defaultrouter="$GATEWAY_IP"

echo "Configuring PF firewall inside $JAIL_NAME..."

JAIL_PF="$JAIL_PATH/etc/pf.conf"

cat <<EOF > "$JAIL_PF"
ext_if = "$EXT_IF"
int_if = "epair0b"
j6_net = "192.168.100.0/24"
j_all  = "10.0.0.0/8"

set skip on lo0
set block-policy drop
set optimization aggressive
set limit { states 20000, frags 5000 }

scrub out on \$ext_if all min-ttl 64

nat on \$ext_if from { \$j_all, \$j6_net } to any -> (\$ext_if)

block in on \$ext_if all

antispoof quick for \$ext_if

pass in quick on \$ext_if inet proto udp from any port 67 to any port 68 keep state

block drop in quick on \$ext_if proto tcp flags FUP/WEUAPRSF
block drop in quick on \$ext_if proto tcp flags /WEUAPRSF
block drop in quick on \$ext_if proto tcp flags SR/SR
block drop in quick on \$ext_if proto tcp flags SF/SF

block in quick on \$ext_if proto tcp from any to any port 22

block drop in quick on \$int_if to (\$int_if)
block drop in quick on \$int_if to (\$ext_if)

block drop in quick inet proto icmp to { (\$ext_if), (\$int_if) } icmp-type echoreq

pass out quick on \$ext_if proto tcp from (\$ext_if) to any flags any keep state
pass out quick on \$ext_if proto { udp, icmp } from (\$ext_if) to any keep state

pass in quick on \$int_if from { \$j_all, \$j6_net } to any keep state
pass out quick on \$int_if to { \$j_all, \$j6_net } keep state
EOF

sysrc -f "$JAIL_RC" pf_enable="YES"
sysrc -f "$JAIL_RC" pf_rules="/etc/pf.conf"

echo "PF configuration installed and enabled in jail."

echo "------------------------------------------------"
echo "INSTALLATION COMPLETE"
echo "Jail Name:  $JAIL_NAME"
echo "Jail IP:    $JAIL_IP"
echo "Gateway:    $GATEWAY_IP (on bridge0)"
echo "------------------------------------------------"
echo "To start the jail: service jail start $JAIL_NAME"
echo "To try DHCP later inside jail: jexec $JAIL_NAME dhclient epair0b"

echo ""
echo "To scan the network, use -scan"
echo ""
jailsec -scan
echo ""
fi
