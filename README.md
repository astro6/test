

#!/bin/bash

# venkateswara.vanka
# uncomment below SWINFO line if Switch info is required
# 04192018 - rhel7 compability
SWINFO="YES"
#
TSTAMP=$(date +%Y%m%d)
HNAME=$(hostname -s)

DUMP_DIR="/tmp/nw"
DUMP_LOG_FILE="${DUMP_DIR}/.SW.log"
SW_LOG_FILE="${DUMP_DIR}/${HNAME}_${TSTAMP}_NW.log"


ITER=1
NET_FIND=1
DUMP_FILE="/tmp/port.log"
SWITCH="ERR"
PORT="ERR"
VLAN="ERR"

##
port_online () {
for ethports in `/sbin/ifconfig -a | egrep "^eth|^etn|^eno|^ens" | awk '{ print $1 }' | awk -F":" '/eth|etn|eno|ens/ { print $1 }'`
do
 /sbin/ifconfig ${ethports} up
done
}

##
chk_dump_dir () {
if [[ ! -d ${DUMP_DIR} ]] ; then
  /bin/mkdir -p ${DUMP_DIR}
fi

> ${DUMP_LOG_FILE}
> ${SW_LOG_FILE}
}

##
hw_info () {
/usr/sbin/dmidecode | grep "System Information" -A4 | egrep -v 'Information|Version:' | awk -F: '{ print $2 }' | paste -s -d ":" >> ${SW_LOG_FILE}

if [[ $SWINFO == YES ]] ; then
  echo "========================================================================================================================================" >> ${SW_LOG_FILE}
  echo "Nic_Port: Switch: SW_Port: VLAN: Vendor: Slot/Bus: Link_Speed: Mac_addr: IP_Addr: Bond_mode:" | column -t >> ${DUMP_LOG_FILE}
  echo "========  ======= =======  ====  ======  ========  ==========  ========  =======  ========="  | column -t >> ${DUMP_LOG_FILE}
else
  echo "==========================================================================" >> ${SW_LOG_FILE}
  echo "Nic_Port: Vendor: Slot/Bus: Link_Speed: Mac_addr: " | column -t >> ${DUMP_LOG_FILE}
  echo "========  ======  ========  =========== ========= " | column -t >> ${DUMP_LOG_FILE}
fi
}
##
bonding_active_port () {
grep -l -w "Slave Interface: $1" /proc/net/bonding/*  2>&1 >/dev/null
if [[ $? == 0 ]] ; then
  ACT_PORT=$(basename `grep -l -w "Slave Interface: ${1}" /proc/net/bonding/*`)
  grep -l -w "Active Slave: ${1}" /proc/net/bonding/* 2>&1 >/dev/null
  if [[ $? == 0 ]] ; then
    get_IP_info ${ACT_PORT}
        ACT_PORT="${ACT_PORT} Active"
  else
    IP_INFO="N/A"
        ACT_PORT="${ACT_PORT} Passive"
  fi
else
  ACT_PORT="N/A"
fi
}

find_slot () {

BUS_ADDR=$(systool -c net $1 | tail -3 | awk '{ print $3 }' | grep -v "^$" | sed 's/"//g')
BUS_ADDR1=$(echo "${BUS_ADDR}" | cut -c1-10)

SLOT_LOC=$(/usr/sbin/dmidecode -t slot | grep -B10 "${BUS_ADDR1}" | awk '/Designation:/ { print $NF}')
if [[ "${SLOT_LOC}" == "" ]] ; then
  SLOT_LOC=${BUS_ADDR}
else
  PCI_LENGTH=$(/usr/sbin/dmidecode -t slot | grep -B10 "${BUS_ADDR1}" | awk '/Length:/ { print $NF}')
  SLOT_LOC="${SLOT_LOC}(${PCI_LENGTH})"
fi

}

##
tcp_dump () {
ETH=$1
ITER=$2

SWITCH="" ; PORT=""; VLAN=""

> ${DUMP_FILE}

nohup /usr/sbin/tcpdump -nn -v -i ${ETH} -s 1500 -c ${ITER} '(ether[12:2]=0x88cc or ether[20:2]=0x2000)' > ${DUMP_FILE} 2>&1 &
TCP_JOBID="$!"

safe_kill ${TCP_JOBID}

if [[ ${JOB_STAT} == 100 ]] ; then

  SWITCH=$(egrep "Device-ID|System Name TLV" ${DUMP_FILE} | awk '{ print $NF }' | awk -F"." '{ print $1 }' | sed "s/'//g"| awk -F '(' '{ print $1 }')
  PORT=$(egrep "Port-ID|Subtype Interface Name|Port Description|Subtype Local" ${DUMP_FILE} | egrep -i "eth|etn|eno|ens" | head -1 | awk '{ print $NF }' | sed "s/'//g" | sed "s/Ethernet/Eth/g")
  #PORT=$(egrep "Port-ID|Subtype Interface Name|Port Description" ${DUMP_FILE} | grep -i ethernet | awk '{ print $NF }' | sed "s/'//g")
  VLAN=$(egrep "VLAN ID|PVID" ${DUMP_FILE} | awk -F":" '{ print $NF }')

     if [[ ${SWITCH} != "" ]] ; then
           NET_FIND=0
         else
           SWITCH="ERR" ; PORT="ERR" ; VLAN="ERR"
         fi
else
  SWITCH="ERR" ; PORT="ERR" ; VLAN="ERR"
fi

}

##
find_eth_ports () {

PORTS=$(/sbin/ifconfig -a | egrep "^eth|^etn|^eno|^ens" | awk '{ print $1 }' | awk -F":" '{ print $1 }')
}


##
get_IP_info () {
IP_INFO=$(/sbin/ifconfig $1 | grep -w inet | awk '{ print $2 }' | sed 's/addr://g')
if [[ ${IP_INFO} == "" ]] ; then
   IP_INFO="No_IP_Set"
fi
}

##
collect_info () {

if [[ $SWINFO == YES ]] ; then
     echo "${line} ${SWITCH%%.*} ${PORT} ${VLAN} ${ETH_VEND} ${SLOT_LOC} ${SPEED} ${MAC_ADDR} ${IP_INFO} ${ACT_PORT}" | column -t >> ${DUMP_LOG_FILE}
else
     echo "${line} ${ETH_VEND} ${SLOT_LOC} ${SPEED} ${MAC_ADDR} " | column -t >> ${DUMP_LOG_FILE}
fi

}

##
find_net_vendor () {

ETHER_BUS_ADDR=$(/usr/sbin/ethtool -i $1 | awk '/^bus-info:/ { print $2 }' | sed 's/^0000://g')
ETH_VEND=$(/sbin/lspci | grep "^${ETHER_BUS_ADDR}" | awk -F: '{ print $3 }' | awk '{ print $1 }')
}

##

find_vlan () {

for line in $( echo ${PORTS})
do
   find_slot ${line}
   MAC_ADDR=$(ethtool -P $line | awk '{ print $NF }' | tr [:lower:] [:upper:])
   #MAC_ADDR=$(cat /sys/class/net/${line}/address)
   if [[ $SWINFO == YES ]] ; then get_IP_info ${line} ; fi
   find_net_vendor ${line}

     if [[ $(cat /sys/class/net/${line}/operstate) == "up" ]] ; then
              bonding_active_port ${line}

                    if [[ $SWINFO == YES ]] ; then tcp_dump ${line} ${ITER} ; fi
                    find_eth_speed ${line}
                    collect_info
                    ACT_PORT=""
          else
       grep "^ONBOOT=yes" /etc/sysconfig/network-scripts/ifcfg-${line} 2>&1 >/dev/null
       if [[ $? == 0 ]] ; then
         SWITCH="ERR"
         PORT="ERR"
         VLAN="ERR"
         SPEED="No_Link"
       else
               SWITCH="No_Link"
               PORT="N/A"
               VLAN="N/A"
               SPEED="N/A"
       fi

             if [[ $SWINFO == YES ]] ; then
                echo "${line} ${SWITCH%%.*} ${PORT} ${VLAN} ${ETH_VEND} ${SLOT_LOC} ${SPEED} ${MAC_ADDR} ${IP_INFO} ${ACT_PORT}" | column -t >> ${DUMP_LOG_FILE}
             else
                 echo "${line} ${ETH_VEND} ${SLOT_LOC} ${SPEED} ${MAC_ADDR} " | column -t >> ${DUMP_LOG_FILE}
             fi
     fi

done
}

##

safe_kill () {

wait_period=100
chk_interval=6
kill_delay=3

while ((wait_period > 0)) ;
do
  sleep ${chk_interval}
  kill -0 $1 2>/dev/null
  if [[ $? == 0 ]] ; then
     ((wait_period -= chk_interval))
  else
     JOB_STAT="100"
     return
  fi
done

{ kill $1 ; sleep ${kill_delay} ; kill -0 $1 2>/dev/null && { kill $1;} ; }
JOB_STAT=101

}


##
empty_line () {
echo >> ${DUMP_LOG_FILE}
}

##
find_eth_speed () {
SPEED=$(/sbin/ethtool $1 | grep Speed: | awk '{ print $2 }')
}

clean_tmp_files () {

\rm ${DUMP_LOG_FILE} ${SW_LOG_FILE}
if [[ $SWINFO == YES ]] ; then \rm ${DUMP_FILE} ; fi
rmdir ${DUMP_DIR}

}

######################## MAIN #####################

#port_online
chk_dump_dir
hw_info
find_eth_ports
find_vlan

cat ${DUMP_LOG_FILE} | /usr/bin/column -t >> ${SW_LOG_FILE}
if [[ $SWINFO == YES ]] ; then
  echo "========================================================================================================================================" >> ${SW_LOG_FILE}
  echo "--------------------------------------------------- `hostname -s` -------------------------------------------------------------------------"
  cat ${SW_LOG_FILE}
  echo "----------------------------------------------------------------------------------------------------------------------------------------"
else
  echo "==========================================================================" >> ${SW_LOG_FILE}
  echo "----------------------------------- `hostname -s` ----------------------------"
  cat ${SW_LOG_FILE}
  echo "--------------------------------------------------------------------------"
fi

clean_tmp_files



