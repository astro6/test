
#!/bin/bash

#
# venkateswara.vanka@nex.com
# 09-28-2015
# system health/build checks
#

# FILL PARAM to run specific module(s)
if [[ $# -lt 1 ]] ; then
# ------ choose individual check - like for Network only "CHK_NTWK" ..... -----------------------------------
#PARAM="CHK_UPTIME"
#
## ------  REBOOT/ BAU checks  -----------
#
PARAM="CHK_UPTIME CHK_HT CHK_CSTATE CHK_LLTB CHK_RAID CHK_NTWK CHK_GW CHK_ROUTES CHK_MOUNTS CHK_SPACE CHK_LTIME CHK_NTP CHK_SVCS"
#
## ------ BUILD CHECKS -------------------
#
#PARAM="CHK_BUILD_FILE CHK_DNS CHK_LLTB CHK_CSTATE CHK_HT CHK_HPFW CHK_RAID CHK_SF CHK_NTWK CHK_GW CHK_ROUTES CHK_YUM_UPD CHK_MOUNTS CHK_SPACE CHK_LTIME CHK_NTP CHK_SVCS CHK_QLEN CHK_VMTOOL CHK_ORA12"
#
##
else
    PARAM=$1
fi
# ----------------------------------------------------------------------------------------------
##############################  GLOBAL VARIABLES SECTION  #################################

# color functions

echor () {
marker='-------------------------------------------------------------------------'
tput -Txterm setf 4;printf "%s %s [ ERROR ]\n" "$1" "${marker:${#1}}";tput -Txterm sgr 0
}
echoa () {
marker='-------------------------------------------------------------------------'
tput -Txterm setf 6;printf "%s %s [ WARN  ]\n" "$1" "${marker:${#1}}";tput -Txterm sgr 0
}
echog () {
marker='-------------------------------------------------------------------------'
tput -Txterm setf 2;printf "%s %s [  OK   ]\n" "$1" "${marker:${#1}}";tput -Txterm sgr 0
}
echob () {
marker='-------------------------------------------------------------------------'
tput -Txterm setf 3;printf "%s %s [ INFO  ]\n" "$1" "${marker:${#1}}";tput -Txterm sgr 0
}
echobg () {
marker='-------------------------------------------------------------------------'
tput -Txterm setf 5;printf "%s %s [ HIGH  ]\n" "$1" "${marker:${#1}}";tput -Txterm sgr 0
}
echop () {
tput -Txterm setf 1 ; echo "$1 -------------------------------------------------------------------" ; tput -Txterm sgr 0
}
summary () {

echo " "
marker="_________________________________________________________________________________________"

if [[ $2 == "OK" ]] ; then
    echo $marker
    tput -Txterm setf 2
    printf "%-29s\t%s\n" "${1}:" "${2}"
    tput -Txterm sgr 0
    echo $marker
elif [[ $2 == "ERROR" ]] ; then
    echo $marker
    tput -Txterm setf 4
    printf "%-29s\t%s\n" "${1}:" "${2}"
    tput -Txterm sgr 0
    echo $marker
elif [[ $2 == "WARN" ]] ; then
    echo $marker
    tput -Txterm setf 6
    printf "%-29s\t%s\n" "${1}:" "${2}"
    tput -Txterm sgr 0
    echo $marker
elif [[ $2 == "NA" ]] ; then
    echo $marker
    tput -Txterm setf 3
    printf "%-29s\t%s\n" "${1}:" "${2}"
    tput -Txterm sgr 0
    echo $marker
else
    echo $marker
    printf "%-29s\t%s\n" "${1}:" "${2}"
    echo $marker
fi

}
#
chk_cmd () {

which $1 >/dev/null
if [[ $? != 0 ]] ; then
 echor "$1 NOT accessible"
fi

}
#

for oscmd in hostname lsb_release date cut awk sort uniq paste tr wget nc
do
 chk_cmd $oscmd
done

#

TS=$(date +%Y%m%d_%H%M)
STOR_DIR="/tmp/maintenance/${TS}"
echo "Run_Time: ${TS}"
SERVER=$(hostname -s)
DOMAIN=$(hostname | cut -d"." -f2- )
OSREL=$(lsb_release -rs)
OSVER=$(echo ${OSREL} | cut -d. -f1)
if [[ ${OSVER} == 6 ]] ; then
  BUILD_FILE="/etc/ICAP-build.info"
else
  BUILD_FILE="/etc/IEBFISA-build.info"
fi
#
exit_fun () {  echo "ERROR: $1" ; exit 100; }

local_repo_bkp () {

env=`hostname -s|cut -c1-3`
env2=`hostname -s|cut -c1-4`

case $env in

  hbr|eds|u0*)  REPO='localrepo-us-lab'
        if [[ $SERVER == eds* ]] ; then
         SITE=EDS
        else
         SITE="SECLAB"
        fi
        if [[ ${OSVER} == 7 ]] ; then PUP="puppet-global-lab";else PUP="puppet-us-lab" ; fi
        REG="US"
        RSLVCONF="nameserver 10.1.143.130
nameserver 10.1.143.241
search if.icap.com
options timeout:1 attempts:1"
        MASTERBK=u0pingbk01p;;

  sec|u1*)  REPO='localrepo-us-prod'
        SITE="SEC"
        if [[ ${OSVER} == 7 ]] ; then PUP="puppet-us-dr";else PUP="puppet-us" ; fi
        REG="US"
        RSLVCONF="nameserver 172.31.91.37
nameserver 172.31.91.103
search if.icap.com
options timeout:1 attempts:1"
        MASTERBK=sec1gbk01p;;

  tky|u3*)
        if [[ $env == tky ]] ; then SITE=TKY
        else
             SITE=SECLAB
        fi
        REPO='localrepo-us-prod'
        if [[ ${OSVER} == 7 ]] ; then PUP="puppet-us-dr";else PUP="puppet-us" ; fi
        REG="US"
        RSLVCONF="nameserver 172.25.24.81
nameserver 172.25.20.103
search if.icap.com
options timeout:1 attempts:1"
        MASTERBK=sec1gbk01p;;

  sti)  REPO='localrepo-us-prod'
        SITE="STI"
        if [[ ${OSVER} == 7 ]] ; then PUP="puppet-us-dr";else PUP="puppet-us" ; fi
        REG="US"
        RSLVCONF="nameserver 172.31.104.37
nameserver 172.31.104.103
search if.icap.com
options timeout:1 attempts:1"
        MASTERBK=sti2gbk01p;;

  prk)  REPO='localrepo-eu-prod'
        SITE="PRK"
        if [[ ${OSVER} == 7 ]] ; then PUP="puppet-eu-dr";else PUP="puppet-eu" ; fi
        REG="EU"
        RSLVCONF="search if.icap.com
options timeout:1
nameserver 172.31.31.73
nameserver 172.31.31.103"
        MASTERBK=prk2gbk01p;;

  slo)  REPO='localrepo-eu-prod'
        SITE="SLO"
        if [[ ${OSVER} == 7 ]] ; then PUP="puppet-eu-dr";else PUP="puppet-eu" ; fi
        REG="EU"
        RSLVCONF="search if.icap.com
options timeout:1
nameserver 172.31.23.73
nameserver 172.31.23.103"
        MASTERBK=slo1gbk01p;;

  e0*)  REPO='localrepo-eu-lab'
        SITE="SLOLAB"
        if [[ ${OSVER} == 7 ]] ; then PUP="puppet-global-lab";else PUP="puppet-us-lab" ; fi
        REG="EU"
        RSLVCONF="nameserver 100.115.80.158
nameserver 100.115.80.159
search if.icap.com
options timeout:1 attempts:1"
        MASTERBK=u0pingbk01p;;

esac

#fi

#if [[ -z ${REPO} ]] ; then  exit_fun "REPO NOT set - REPO:${REPO}" ; fi
echo

}

status_tag () {
 tput -Txterm sgr 0
 echo
 echo "IEBFISA $1 $2 $3"
 echo
}

create_dump_dir () {
mkdir -p ${STOR_DIR}
cd ${STOR_DIR}
}

find_platform () {
PLATFORM=$(/usr/sbin/dmidecode -s system-manufacturer | awk '!/#/ { print $1 }' | sed 's/,//g')
if [[ ${PLATFORM} == "Red" ]] ; then
 PLATFORM="KVM"
fi

status_tag HW ${PLATFORM} ${PLATFORM}
}

low_lat () {
LOWLAT=$( awk '/LOWLAT/ { print $2 }' ${BUILD_FILE} )
}


show_glob_vars () {

GLOB_VARS="SERVER DOMAIN PLATFORM MODEL SNO ILOIP SOCKETS CORES MEMORY OSVER OSREL REG SITE LOWLAT REPO PUP MASTERBK STOR_DIR"
GLOB_FUN="exit_fun status_tag show_glob_vars echor echoa echog echob echobg top_b bot_b echop"
echob "Global Variables"
    for i in `echo ${GLOB_VARS}`
    do
        printf "%-10s\t%s\n" "$i:" "${!i}"
        if [[ ${!i} == "" ]] ; then echor "Variable $i not being set properly ";fi
    done
echob "-"

}
top_b () {
echo
echop "Running $1 "
}
bot_b () {
echo
echop "End of $1 "
}
#
model_sno () {

if [[ ${PLATFORM} == "HP" ]] ; then
    chk_cmd /usr/bin/ipmitool
    MODEL=$(/usr/sbin/dmidecode -s  system-product-name | awk '!/#/ { print }' | sed 's/ //g')
    SMODEL=$(echo ${MODEL} | sed 's/ProLiant//g')
    SNO=$(/usr/sbin/dmidecode -s  system-serial-number | awk '!/#/ { print $1 }')
    SOCKETS=$(cat /proc/cpuinfo | grep  'physical id' | sort -u | wc -l)
    CORES=$(cat /proc/cpuinfo | awk '/cpu cores/ { print $4 }' | sort -u)
    MEMORY=$(free -m | awk '/Mem:/ { print $2 }')
    #MEMORY=$(/usr/local/bin/memconf | awk '/^total memory/ { print  $4 }')
    ILOIP=$(/usr/bin/ipmitool lan print | awk '/^IP Address              :/ { print $4 }')

elif [[ ${PLATFORM} == "VMware" ]] ; then
    MODEL=VM
    SMODEL=${PLATFORM}
    SNO="N/A"
    ILOIP="N/A"
    SOCKETS=$(cat /proc/cpuinfo | grep  'physical id' | sort -u | wc -l)
    CORES=$(cat /proc/cpuinfo | awk '/cpu cores/ { print $4 }' | sort -u)
    MEMORY=$(free -m | awk '/Mem:/ { print $2 }')
    #MEMORY=$(/usr/local/bin/memconf | awk '/^total memory/ { print  $4 }')

elif [[ ${PLATFORM} == "KVM" ]] ; then
    MODEL=KVM
    SMODEL=${PLATFORM}
    SNO="N/A"
    ILOIP="N/A"
    SOCKETS="N/A"
    CORES=$(cat /proc/cpuinfo | grep -c '^processor' )
    MEMORY=$(free -m | awk '/Mem:/ { print $2 }')
    #MEMORY=$(/usr/local/bin/memconf | awk '/^total memory/ { print  $4 }')

fi

status_tag MODEL ${SMODEL} ${SMODEL}
status_tag S/N ${SNO} ${SNO}
status_tag OS ${OSREL} ${OSREL}

}

# -------------------------------------------------------------------
CHK_INF_SRV () {
# -------------------------------------------------------------------
CPUP=$(getent hosts ${PUP} | awk '{ print $1 }')
if [[ ${CPUP} != "" ]] ; then
    echob "Checking puppet master \"${CPUP}\" "
    #nmap -p 8140 ${CPUP} | grep ^8140 | grep open
    nc -w 2 -z -v ${CPUP} 8140
    pupstat1=$(echo $?)
    if [[ $pupstat1 != 0 ]] ; then
        CPUP=puppet
        echob "Checking puppet master \"${CPUP}\" "
        #nmap -p 8140 ${CPUP} | grep ^8140 | grep open
        nc -w 2 -z -v ${CPUP} 8140
        pupstat1=$(echo $?)
    fi
else
    CPUP=puppet
    echob "Checking puppet master \"${CPUP}\" "
    #nmap -p 8140 ${CPUP} | grep ^8140 | grep open
    nc -w 2 -z -v ${CPUP} 8140
    pupstat1=$(echo $?)
fi
#nc -w 2 -z -v ${CPUP} 443
pupstat2=0
#pupstat2=$(echo $?)
[ ${pupstat1} -ne 0 -o ${pupstat2} -ne 0 ] && echor "Puppet master ${PUP}/${CPUP} NOT reachable "
#
CREPO=$(getent hosts ${REPO} | awk '{ print $1 }')
echob "Checking REPO server ${REPO}/${CREPO} "
#nmap -p 80 ${CREPO} | grep ^80 | grep open
nc -w 2 -z -v ${CREPO} 80
if [[ $? == 0 ]] ; then
    REPO_STAT=OK
else
    echob "Checking REPO as 192.168.8.2 "
    #nmap -p 80 192.168.8.2 | grep ^80 | grep open
    nc -w 2 -z -v 192.168.8.2 80
    if [[ $? == 0 ]] ; then
        REPO_STAT=OK
        REPO=192.168.8.2
        PUP=192.168.8.97
    else
        echor "Local Repo ${REPO}/${CREPO}/192.168.8.2 are NOT reachable "
    fi
    if [[ -z ${REPO} ]] ; then  echor "Local REPO NOT set " ; fi
fi
#
}


find_platform
model_sno
local_repo_bkp
low_lat
show_glob_vars
CHK_INF_SRV
############################################ END OF GLOBAL VARIABLES SECTION ####################################

# -------------------------------------------------------------------
CHK_UPTIME () {
# -------------------------------------------------------------------

up_time=$(awk '{printf("%dd:%02dh:%02dm",($1/60/60/24),($1/60/60%24),($1/60%60))}' /proc/uptime)
last_boot=$(who -b | awk '{ print $3"_"$4 }')
echo "---------- Host is UP since ${up_time} and last reboot was at ${last_boot} ----------"
status_tag UPTIME NA ${up_time}
status_tag LASTBOOT NA ${last_boot}

}
# -------------------------------------------------------------------
CHK_CSTATE () {
# -------------------------------------------------------------------

CSTATE="OK"

if ! [[ "${PLATFORM}" == "KVM" || "${PLATFORM}" == "VMware" ]] ; then
    if [[ $(cat /sys/module/processor/parameters/max_cstate 2>&1) == 0 || $(cat /sys/module/intel_idle/parameters/max_cstate 2>&1) == 0 ]] ; then
        echog "CSTATE status "
    else
        echor "CSTATE not set "
        CSTATE=ERROR
    fi

    if ! [[ ${OSVER} == "5" ]] ; then
        echo "cat cmdline contents "
        cat /proc/cmdline
        for grub in idle=poll intel_idle.max_cstate=0 mce=ignore_ce nosoftlockup
        #for grub in processor.max_cstate=1 idle=poll intel_idle.max_cstate=0 nosoftlockup
        do
            grep $grub /proc/cmdline > /dev/null
            if [[ $? != 0 ]] ; then
                echor "$grub NOT set in grub.conf "
                 CSTATE=ERROR
            fi
        done
        if [[ ${LOWLAT} == "Y" ]] ; then
          if [[ ${OSVER} == 6 ]] ; then
            [ $(rpm -qa | grep icap-lowlat >/dev/null 2>&1 ; echo $?) == 0 ] || echor "icap-lowlat rpm NOT installed "
            [[ $(/etc/init.d/icap-lowlat status| egrep -i "rx|tx" |egrep -v '0|off') != "" ]] && echor "Coalescence settings NOT in place "
          else
            [ $(rpm -qa | grep iebfisa-lowlat >/dev/null 2>&1 ; echo $?) == 0 ] || echor "iebfisa-lowlat rpm NOT installed "
            [[ $(/usr/local/bin/iebfisa-lowlat status | egrep -i "rx|tx" |egrep -v '0|off') != "" ]] && echor "Coalescence settings NOT in place "
          fi
        fi
    fi

else

    echob "Skipping ${PLATFORM} platform CPU CSTATE checks ....."
    CSTATE=NA

fi
summary "CSTATE/grub.conf " "${CSTATE}"
status_tag CSTATE ${CSTATE} ${CSTATE}


}
######
# -------------------------------------------------------------------
CHK_HT () {
# -------------------------------------------------------------------

if ! [[ ${PLATFORM} == "KVM" || ${PLATFORM} == "VMware" ]] ; then

    HTSTAT="OK"
    #
    CURHT=$(cat /proc/cpuinfo | grep -E "cpu cores|siblings|physical id" |xargs -n 11 |sort -u)
    #
    while read line
    do
        set -- $line
        if [[ $7 != ${11} ]] ; then
            HTSTAT="ERROR"
            echor "Hyperthread setting not proper "
        fi
    done < <(echo $CURHT)

    echo -e "\n${CURHT}"

    summary "Hyperthread " "${HTSTAT}"


else
    echob "Skipping ${PLATFORM} Platform ........."
    HTSTAT=NA

fi

status_tag HT ${HTSTAT} ${HTSTAT}

}
######

# -------------------------------------------------------------------
CHK_HPFW () {
# -------------------------------------------------------------------

find_cur_fws () {
chk_cmd ipmitool
BIOS_CUR=$(/usr/sbin/dmidecode -s bios-release-date | awk '!/#/ { print $1 }')
ILOFW_CUR=$(/usr/bin/ipmitool mc info | awk '/Firmware Revision/ { print $4 }' | head -1)
[ -f /usr/sbin/ssacli ] && hp_arr_cmd=/usr/sbin/ssacli || hp_arr_cmd=hpssacli
chk_cmd $hp_arr_cmd
ARRAY_CUR=$($hp_arr_cmd controller all show config detail | awk '/^   Firmware Version:/ { print $NF }' | head -1)

}
#
cmp_fws () {

for fw in BIOS ILOFW ARRAY
do
    export CHK_STAT=${fw}_STAT CUR=${fw}_CUR EXP=${fw}_EXP
    if [[ ${!CUR} == ${!EXP} ]] ; then
        echog "current $fw ver ${!CUR}  is same as expected $fw ver ${!EXP} "
        CHK_STAT=OK
    else
                if [[ $fw == BIOS ]] ; then
                        bcur=$(date -d ${!CUR} +%s)
                        bexp=$(date -d ${!EXP} +%s)
                    cmp_stat=$(echo "${bcur} < ${bexp}" | bc)
                    if [[ $cmp_stat == 1 ]] ; then
                                CHK_STAT=ERROR
                                echor "current $fw ver ${!CUR} is LOWER than expected ver ${!EXP} "
                        else
                                CHK_STAT=HIGH
                                echobg "current $fw ver ${!CUR} is HIGHER than expected ver ${!EXP} "
                        fi
                else
                    cmp_stat=$(echo "${!CUR} < ${!EXP}" | bc)
                        if [[ $cmp_stat == 1 ]] ; then
                                CHK_STAT=ERROR
                                echor "current $fw ver ${!CUR} is LOWER than expected ver ${!EXP} "
                        else
                                CHK_STAT=HIGH
                                echobg "current $fw ver ${!CUR} is HIGHER than expected ver ${!EXP} "
                        fi


                fi
    fi

    status_tag ${fw} ${CHK_STAT} ${!CUR}
done


}
#
kvm_vmware_fw () {

echob "Skipping ${PLATFORM} platform ........."

for i in BIOS ILOFW ARRAY
do
    status_tag $i NA NA
done

}
#
#######
if ! [[ ${PLATFORM} == "KVM" || ${PLATFORM} == "VMware" ]] ; then
    #FW_REF="http://${REPO}/scripts/${PLATFORM}/${MODEL}/${OSREL}/FW_INFO.txt"
    FW_REF="http://${REPO}/scripts/${PLATFORM}/FW_INFO.txt"
    #echo ${FW_REF}
    if [[ ${PLATFORM} == "HP" ]] ; then
        chk_cmd wget
        wget --waitretry=5 --tries=2 --timeout=10 -q ${FW_REF}
        if [[ $? -ne 0 ]] ; then
            echor "wget ${FW_REF} failed/not exist "
            BIOS_EXP=ERR ; ILOFW_EXP=ERR ; ARRAY_EXP=ERR
        else
            #set -- $(cat FW_INFO.txt | grep -v "#")
            set -- $(cat FW_INFO.txt | grep -w "^${MODEL}" | grep rhel${OSVER})
            BIOS_EXP=$2 ; ILOFW_EXP=$3 ; ARRAY_EXP=$4
            echob "${MODEL}:  Expected BIOS: ${BIOS_EXP}  ILO: ${ILOFW_EXP}  ARRAY_FW: ${ARRAY_EXP}"
        fi
        find_cur_fws
        cmp_fws
    else
        echob "Skipping ${PLATFORM} platform ..............."
        kvm_vmware_fw
    fi
else
    kvm_vmware_fw
fi

}

# -------------------------------------------------------------------
CHK_NTWK () {
# -------------------------------------------------------------------

ifcfg_dir="/etc/sysconfig/network-scripts"
if [[ ${OSVER} == 6 ]] ; then
 bond0_exp="eth0 eth2"
 bond1_exp="eth1 eth3"
 bond2_exp="eth4 eth5"
 bond4_exp="eth6 eth7"
else
 bond0_exp="etn0 etn2"
 bond1_exp="etn1 etn3"
 bond2_exp="etn4 etn5"
 bond4_exp="etn6 etn7"
fi
NW_STAT=OK
#
echo "-------- ip ad out ----------"
/sbin/ip ad
#
get_active_ports () {

eth_ports=$(egrep -l -i "^onboot=yes" ${ifcfg_dir}/ifcfg-e{th,tn,n}* 2>/dev/null | awk -F"-" '{ print $NF }' | sort | paste -s )
#br_ports=$(egrep -l -i "^onboot=yes" ${ifcfg_dir}/ifcfg-br* | awk -F"-" '{ print $NF }' | sort | paste -s)
bond_ports=$(egrep -l -i "^onboot=yes" ${ifcfg_dir}/ifcfg-bond* 2>/dev/null | awk -F"-" '{ print $NF }' | sort | paste -s )
}

find_os_ports () {
OS_PORTS=$(/sbin/ifconfig -a | egrep "^eth|^etn|^eno|^ens" | awk '{ print $1 }' | awk -F":" '{ print $1 }')
for osport in ${OS_PORTS}
do
 if ! [[ -f /etc/sysconfig/network-scripts/ifcfg-$osport ]] ; then echor "/etc/sysconfig/network-scripts/ifcfg-$osport not exist ";fi
done
}

find_exp_speed () {

chk_cmd /sbin/ethtool
if [[ `/sbin/ethtool -i $1 | awk '/driver:/ { print $NF }'` == "sfc" ]] ; then
    /sbin/ethtool -m $1 | grep 'Ethernet: 1000BASE-T' > /dev/null
    if [[ $? == 0 ]] ; then
        speed_exp="1000Mb/s"
    else
        speed_exp="10000Mb/s"
    fi
        PTYPE="SFC"
else
    grep 'speed 100 duplex' /etc/sysconfig/network-scripts/ifcfg-$1 >/dev/null 2>&1
    if [[ $? == 0 ]] ; then
        speed_exp="100Mb/s"
    else
        speed_exp="1000Mb/s"
        PTYPE=""
    fi
fi

}

#
get_active_ports

if ! [[ "${eth_ports}" == "" ]]; then
        for eth in ${eth_ports}
        do
                if ! [[ "${PLATFORM}" == "KVM" || "${PLATFORM}" == "VMware" ]] ; then
                        find_exp_speed $eth
                        set -- $(/sbin/ethtool $eth | awk '/Speed:|Duplex:|Link detected:/ { print $NF }' | paste -s )
                        echo "${eth}: ${PTYPE} Speed=$1 Duplex=$2 Link=$3 "
                        if ! [[ $1$2$3 == "${speed_exp}Fullyes" ]] ; then
                                NW_STAT=ERROR
                                echor "port $eth Speed/Duplex/Link ERROR"
                        fi
                else
                        LINK=$(/sbin/ethtool $eth | awk '/Link detected:/ { print $NF }')
                        if ! [[ ${LINK} == "yes" ]] ; then
                                echor "$eth link is down - ERROR"
                                NW_STAT=ERROR
                        else
                                echo "$eth Link status is - ${LINK}  - OK"
                        fi
                fi

        done
else
        echor "No active eth ports "
        NW_STAT=ERROR
fi

#
find_os_ports
#
if ! [[ "${bond_ports}" == "" ]]; then
        for bond in ${bond_ports}
        do
                export SLA_EXP=${bond}_exp
                SLA_CUR=$(egrep "Slave Interface:" /proc/net/bonding/${bond} | awk '{ print $NF }' | sort | paste -s -d ' ' )
                #SLA_CUR=$(egrep "Slave Interface:" /proc/net/bonding/${bond} | awk '{ print $NF }' | sort | paste -s | tr -s '\t' ' ' )
                MAS_CUR=$(egrep "Currently Active Slave:" /proc/net/bonding/${bond} | awk '{ print $NF }')
                MAS_EXP=$(set -- ${!SLA_EXP} ; echo $1)
                if ! [[ ${SLA_CUR} == ${!SLA_EXP} ]] ; then
                        echor "$bond current configured slaves are ${SLA_CUR}, it should be ${!SLA_EXP} "
                        NW_STAT=ERROR
                else
                        echo "$bond current configured slavers are ${SLA_CUR} - OK"
                fi
                if ! [[ ${MAS_CUR} == ${MAS_EXP} ]] ; then
                        echor "$bond current primary is ${MAS_CUR}, it should be ${MAS_EXP} "
                        NW_STAT=ERROR
                else
                                for bsla in ${!SLA_EXP}
                                do
                                        LINK=$(/sbin/ethtool $bsla | awk '/Link detected:/ { print $NF }')
                                        if ! [[ ${LINK} == "yes" ]] ; then
                                                echoa "$bond - NO Redundancy, current primary is ${MAS_CUR} "
                                                echor "$bond - $bsla link is down "
                                                BSTAT=WARN
                                        fi
                                done
                        [ "${BSTAT}" != "WARN" ] && echog "$bond current primary is ${MAS_CUR} "
                fi
        done
else
        if [[ "${PLATFORM}" == "KVM" ]] ; then
            echob "Skipping ${PLATFORM} plaform bonding checks  - NA"
        else
            echor "No active bonds configured "
            NW_STAT=ERROR
        fi
fi
#
#
ifcfg_files_chk () {

ports=$(ls -l /etc/sysconfig/network-scripts/ifcfg-{eth,etn,en,bond}* 2>/dev/null| awk -F"-" '{ print $NF }' | paste -s)

for port in $ports
do
 ifconfig $port >/dev/null 2>&1
 if [[ $? != 0 ]] ; then
   echor "ifcfg-$port file exist, OS doesn't see the physcial/logical port"
 fi
done

##
osports=$(systool -c net | awk '/Class Device =/ { print $NF }'  | grep -v lo | sed "s/\"//g" | paste -s)

for osport in $osports
do
 if [[ $osport == bond* ]] ; then
   ifconfig $osport | grep inet >/dev/null 2>&1
   if [[ $? == 0 ]] ; then
       ls -al /etc/sysconfig/network-scripts/ifcfg-$osport > /dev/null
        if [[ $? != 0 ]] ; then
         echor "OS port $osport exist, ifcfg-$osport file doesn't exist"
       fi
   fi
 else
   ifconfig $osport | grep inet >/dev/null 2>&1
   if [[ $? == 0 ]] ; then
       ls -al /etc/sysconfig/network-scripts/ifcfg-$osport > /dev/null
        if [[ $? != 0 ]] ; then
         echor "OS port $osport exist, ifcfg-$osport file doesn't exist"
       fi
   fi
 fi
done

##

sroutes=$(ls -l /etc/sysconfig/network-scripts/ifcfg-route* 2>/dev/null| awk -F"-" '{ print $NF }' | paste -s)
for routes in $sroutes
do
 ifconfig $routes  2 > /dev/null 2>&1
 if [[ $? != 0 ]] ; then
   echor "ifcfg-$routes file exist, OS doesn't see the physcial/logical port"
 fi
done

}
#
##
if [[ ${PLATFORM} == "VMware" ]] ; then
 lsmod | grep ^e1000 2>/dev/null
 if [[ $? == 0 ]] ; then
        echoa "e1000 network driver loaded "
 fi
fi
#
# check if any rename interface exist
ip ad | grep rename
if [[ $? == 0 ]] ; then
  echor "rename interface names exist "
  NW_STAT=ERROR
fi
#
# check if both ifcfg-* file and actual ports on the box match
#
ifcfg_files_chk
#
###############################
network_layout () {
# uncomment below SWINFO line if Switch info is required
#
#SWINFO="YES"
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

  SWITCH=$(egrep "Device-ID|System Name TLV" ${DUMP_FILE} | awk '{ print $NF }' | awk -F"." '{ print $1 }' | sed "s/'//g")
  PORT=$(egrep "Port-ID|Subtype Interface Name|Port Description|Subtype Local" ${DUMP_FILE} | egrep -i "eth|etn|eno|ens" | head -1 | awk '{ print $NF }' | sed "s/'//g")
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
   MAC_ADDR=$(ethtool -P $line | awk '{ print $NF }')
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
}
network_layout

summary "Network " "${NW_STAT}"

status_tag NTWK ${NW_STAT} ${NW_STAT}
}
######
# -------------------------------------------------------------------
CHK_PTP () {
# -------------------------------------------------------------------
#
  STAT=$(find /var/lib/sfptpd/topology -mmin 1 -exec echo "current" \;)
  if [[ $STAT == "current" ]] ; then
    grep grandmaster /var/lib/sfptpd/topology > /dev/null 2>&1
    if [[ $? == 0 ]] ; then
      grep alarm /var/lib/sfptpd/topology > /dev/null 2>&1
      if [[ $? == 0 ]] ; then
         echor "ERROR: ptp in alarm state"
         PTP_STAT=ERROR
      fi
      GM_OFFSET=$(cat /var/lib/sfptpd/topology | egrep -A4 grandmaster | tail -1 )
      OFFSET=$(cat /var/lib/sfptpd/topology | egrep -A4 grandmaster | tail -1 | awk '{ print $1 }' | sed "s/-//g;s/+//g")
      offset=$(echo "$OFFSET > 200.000" | bc )
      if [[ $offset == 0 ]] ;then
        echog "offset with GrandMaster - ${GM_OFFSET}"
        PTP_STAT=OK
      else
        echor "offset with GrandMaster - ${GM_OFFSET}"
        PTP_STAT=ERROR
      fi
    else
      echor "Not binding with PTP GrandMaster "
      PTP_STAT=ERROR
    fi
  else
    echor "PTP in error state "
    PTP_STAT=ERROR
  fi

  cat /var/lib/sfptpd/topology


summary "PTP offset(${GM_OFFSET}) " "${PTP_STAT}"

status_tag NTP ${PTP_STAT} ${offset}

}
# -------------------------------------------------------------------
CHK_NTP () {
# -------------------------------------------------------------------

run_chronyc () {

chronyc sources | grep ^\^\\* >/dev/null 2>&1
if [[ $? != 0 ]] ; then
  echor "not connected to chrony Timeserver "
  NTP_STAT=ERROR ; NTP_MSG=ERROR ; ntpoff="Not_Binding"
else
  chronyc -n sources
  timeserver=$(chronyc -n sources | grep ^\^\\*  | awk '{ print $2 }')
  set -- $(chronyc -n tracking | egrep "^Stratum|^Last offset" | paste -s -d " ")
  ntpoff=$(echo $7 | sed "s/-//g;s/+//g")
  offset=$(echo "$ntpoff > 30.000000000" | bc)
  if [[ $offset == 0 ]] ; then
          NTP_STAT=OK ; NTP_MSG=${timeserver}
  else
          NTP_STAT=WARN ; NTP_MSG=${timeserver}
          echoa "ntp offset is above threshold "
  fi

fi
summary "Chronyc offset(${ntpoff}) " "${NTP_STAT}"

status_tag NTP ${NTP_STAT} ${NTP_MSG}

}
##

if [[ ${OSVER} == 6 ]] ; then
  chkconfig --list ntpd | grep -w on

  if [[ $? == 0 ]] ; then

    chk_cmd /usr/sbin/ntpq
    /usr/sbin/ntpq -p | grep "^\*" >/dev/null

    if [[ $? == 0 ]] ; then
            set -- $(/usr/sbin/ntpq -np | grep "^\*")
            ntpoff=$(echo $9 | sed 's/-//g' )
            offset=$(echo "$ntpoff > 30.000" | bc )
            if [[ $offset == 0 ]] ; then
                  NTP_STAT=OK ; NTP_MSG=$(echo $1 | sed 's/*//')
            else
                  NTP_STAT=WARN ; NTP_MSG=$(echo $1 | sed 's/*//g')
                  echoa "ntp offset is above threshold "
            fi

    else
      NTP_STAT=ERROR ; NTP_MSG=ERROR ; ntpoff="Not_Binding"
      echor "NTP Not_Binding "

    fi
    /usr/sbin/ntpq -np
    summary "NTP offset(${ntpoff}) " "${NTP_STAT}"

    status_tag NTP ${NTP_STAT} ${NTP_MSG}
  else
    service sfptpd status | grep Running > /dev/null 2>&1
    if [[ $? == 0 ]] ; then
      CHK_PTP
    else
      chkconfig --list | egrep "ntp|sfptpd"
      echor "NTP/PTP service is NOT running "
      NTP_STAT=ERROR ; NTP_MSG=ERROR ; ntpoff="Not_Binding"
    fi
  fi
else
  systemctl status chronyd 2>/dev/null| grep 'Active: active (running)'
  if [[ $? == 0 ]] ; then
    run_chronyc
  else
    #systemctl status sfptpd 2>/dev/null| grep 'Active: active (running)'
    service sfptpd status | grep Running > /dev/null 2>&1
    if [[ $? == 0 ]] ; then
      CHK_PTP
    else
      systemctl list-unit-files | egrep "chronyd|sfptpd"
      chkconfig --list | egrep "sfptpd"
      echor "chronyd/PTP service is NOT running "
      NTP_STAT=ERROR ; NTP_MSG=ERROR ; ntpoff="Not_Binding"
    fi
  fi
fi
}
######
# -------------------------------------------------------------------
CHK_LTIME () {
# -------------------------------------------------------------------
/usr/bin/test -h /etc/localtime
if [[ $? == 0 ]] ; then
   LT_LINK=$(ls -lad /etc/localtime | awk -F\/ '{print $NF}')
else
   if [[ ${REG} == US ]] ; then
      CHKSUMA=$(/usr/bin/cksum /usr/share/zoneinfo/America/New_York | awk '{ print $1 }')
      CHKSUMB=$(/usr/bin/cksum /etc/localtime | awk '{ print $1 }')
      LT_LINK="New_York"
      if [[ $CHKSUMA != $CHKSUMB ]] ; then
          echor "etc-localtime and zoneinfo checksum are NOT matching "
      fi
   elif [[ ${REG} == EU ]] ; then
      CHKSUMA=$(/usr/bin/cksum /usr/share/zoneinfo/Europe/London | awk '{ print $1 }')
      CHKSUMB=$(/usr/bin/cksum /etc/localtime | awk '{ print $1 }')
      LT_LINK="London"
      if [[ $CHKSUMA != $CHKSUMB ]] ; then
          echor "etc-localtime and zoneinfo checksum are NOT matching "
      fi
   fi


fi
if [[ ${OSVER} == 6 ]] ; then
  ZONE=$(grep ZONE /etc/sysconfig/clock | grep -v \^#| awk -F= '{print $2}')
else
  ZONE=$(/usr/bin/timedatectl | awk '/Time zone:/ { print $3 }')
fi
ZONE_REG_CUR=$(date '+%Z')
DATE_CUR=$(date +%m%d%H%M%S)
#
find_US_dst_dates () {

for j in {1..7}; do
    smonth=03
    wmonth=11
    SDATE=$(date -d "$smonth/1 + 7 day + $j day" +"%w %m%d")
    WDATE=$(date -d "$wmonth/1 + $j day" +"%w %m%d")
    # day of week (1..7); 1 is Monday
    if [ ${SDATE:0:1} -eq 0 ]; then
        USS=${SDATE:2}
    fi
    if [ ${WDATE:0:1} -eq 0 ]; then
        USW=${WDATE:2}
    fi
done


}
#
find_EU_dst_dates () {

for j in {1..7}; do
    smonth=03
    wmonth=10
    SDATE=$(date -d "$smonth/1 + 1 month - $j day" +"%w %m%d")
    WDATE=$(date -d "$wmonth/1 + 1 month - $j day" +"%w %m%d")
    # day of week (1..7); 1 is Monday
    if [ ${SDATE:0:1} -eq 0 ]; then
        EUS=${SDATE:2}
    fi
    if [ ${WDATE:0:1} -eq 0 ]; then
        EUW=${WDATE:2}
    fi
done


}
#
find_US_dst_dates
find_EU_dst_dates
zone_reg_exp () {

if [[ $1 == "US" ]] ; then
    [ ${DATE_CUR} -ge ${USS}020000 -a ${DATE_CUR} -le ${USW}010000 ]
        if [[ $? == 0 ]] ; then ZONE_REG_EXP="EDT"
    else
        ZONE_REG_EXP="EST"
    fi
elif [[ $1 == "EU" ]] ; then
    [ ${DATE_CUR} -ge ${EUS}010000 -a ${DATE_CUR} -le ${EUW}010000 ]
        if [[ $? == 0 ]] ; then ZONE_REG_EXP="BST"
    else
        ZONE_REG_EXP="GMT"
    fi

fi

}

#

zone_reg_exp ${REG}

if [[ ( ${REG} == "US" ) && ( ${LT_LINK} == "New_York" ) && ( ${ZONE_REG_CUR} == ${ZONE_REG_EXP} ) && ( ${ZONE} == '"America/New_York"'  || ${ZONE} == "America/New_York") ]] ; then
    LTIME_STAT=OK ; LTIME_MSG=${ZONE_REG_CUR}
    echog "Localtime:  Region: ${REG}   ZONE: ${ZONE}   ST: ${ZONE_REG_CUR} "
elif [[ ( ${REG} == "EU" ) && ( ${LT_LINK} == "London" ) && ( ${ZONE_REG_CUR} == ${ZONE_REG_EXP} ) && ( ${ZONE} == '"Europe/London"' || ${ZONE} == "Europe/London" ) ]] ; then
    LTIME_STAT=OK ; LTIME_MSG=${ZONE_REG_CUR}
    echog "Localtime:  Region: ${REG}   ZONE: ${ZONE}   ST: ${ZONE_REG_CUR} "
else
    LTIME_STAT=ERROR ; LTIME_MSG=${ZONE_REG_CUR}
    echor "Localtime: Region: ${REG}   ZONE: ${ZONE}   ST: ${ZONE_REG_CUR} "
fi

status_tag LTIME ${LTIME_STAT} ${LTIME_MSG}

}

# -------------------------------------------------------------------
CHK_GW () {
# -------------------------------------------------------------------
#set -- $(/sbin/ip route | grep default)
chk_cmd /sbin/ip
def_route=$(/sbin/ip route | grep default)

if [[ ${def_route} != "" ]] ; then
    set -- ${def_route}
    gateway=$3 ; inf=$5
 if [[ ${gateway} =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$ ]] ; then
    if [[ $(ping -q -c 1 $gateway | grep -q 100% 2>/dev/null; echo $?) == 0 ]] ; then
        if [[ $(/sbin/arping -I $inf -c 1 $gateway >/dev/null;echo $?) == 0 ]] ; then
            GW_STAT=OK ; GW_MSG="DMZ"
            echob "DMZ or restricted GW in use "
        else
            GW_STAT=ERROR
            echor "Gateway $gateway access using n/w interface $inf "
        fi
    else
        ping -q -c 1 $gateway

        GW_STAT=OK
    fi
 else
    GW_STAT=ERROR; echor "improper gateway record $gateway using n/w interface $inf "
 fi

else
    echor "Default route not exist "
    GW_STAT=ERROR
fi
summary "Gateway $gateway access using n/w interface $inf ${GW_MSG}" "${GW_STAT}"
status_tag GW ${GW_STAT} ${GW_STAT}
}

# -------------------------------------------------------------------
CHK_SF () {
# -------------------------------------------------------------------

#
get_sfc_cur () {

    echo "$port FW details"
    set -- $(echo "${SF_OUT}" | grep -A4 $1 | awk -F: '/Controller version:/ { print $2 }' | sed 's/ //g' | paste -d ' ' -s  )
    #set -- $(echo "${SF_OUT}" | grep -A4 $1 | awk -F: '/Firmware version:|Controller version:|Boot ROM version:/ { print $2 }' | sed 's/ //g' | paste -d ' ' -s  )
    CNT_VER_CUR=$1
    #SF_FW_VER_CUR=$1 ; CNT_VER_CUR=$2 ; ROM_VER_CUR=$3


}
#

find_sf_ports () {

for sfpci in ${SF_PCI}
do
        echo "${SYS_OUT}" | grep -B1 $sfpci | awk '/eth|eno|ens|etn/ { print $NF }' | sed 's/"//g'
done | paste -d" " -s

}

#

chk_ethtool_opts () {
for sfport in ${SF_PORTS}
do
        cfgfile="/etc/sysconfig/network-scripts/ifcfg-${sfport}"
        grep '^ONBOOT=yes'  $cfgfile >/dev/null 2>&1
        if [[ $? == 0 ]] ; then
           /sbin/ethtool -m ${sfport} | grep 'Ethernet: 1000BASE-T' > /dev/null
           if [[ $? != 0 ]] ; then
                grep -w '^ETHTOOL_OPTS="autoneg off speed 10000 duplex full"' $cfgfile >/dev/null 2>&1
                if [[ $? == 0 ]] ; then
                        echog "ifcfg-$sfport has been set with ETHTOOl_OPTS "
                        autoneg_stat=$(ethtool $sfport | grep Auto-negotiation | awk '{ print $NF }')
                        if [[ $autoneg_stat != "off" ]] ; then
                                echor "Auto-negotiation not active on $sfport "
                                SF_STAT=ERROR ; SF_MSG=ERROR
                        fi
                else
                        echor "ifcfg-$sfport has NOT been set with ETHTOOl_OPTS "
                        SF_STAT=ERROR ; SF_MSG=ERROR
                fi
           else
             echob "${sfport}  - SFC port with Ethernet: 1000BASE-T connector "
           fi

        fi
done
}
#
cmp_sfc_fws () {

for sfchk in ONLOAD CNT_VER
do
    if [[ ${ONLOAD_CHK} == "done" && ${sfchk} == ONLOAD ]] ; then continue ; fi
    export CUR=${sfchk}_CUR EXP=${sfchk}_EXP
    if [[ ${!CUR} == ${!EXP} ]] ; then
        echog "current $sfchk ver ${!CUR}  is same as expected $sfchk ver ${!EXP} "
    else
        MCUR=$(echo ${!CUR} | cut -d"." -f 1-3 | sed 's/v//g;s/\.//g')
        MEXP=$(echo ${!EXP} | cut -d"." -f 1-3 | sed 's/v//g;s/\.//g')
        if [[ ${MCUR} == ${MEXP} ]] ; then
                echog "$sfchk MAJOR ver of ${!CUR} is same as expected MAJOR ver ${!EXP} "
        else
                sf_cmp_stat=$(echo "${MCUR} < ${MEXP}" | bc)
                if [[ $sf_cmp_stat == 1 ]] ; then
                        SF_STAT=ERROR ; SF_MSG=ERROR
                        echor "$sfchk MAJOR ver ${!CUR} is LOWER than expected MAJOR ver ${!EXP} "
                else
                        SF_STAT=HIGH ; SF_MSG=HIGH
                        echobg "$sfchk MAJOR ver ${!CUR} is HIGHER than expected MAJOR ver ${!EXP} "
                fi
        fi
    fi
done
}
#
kvm_vmware_sfc_fw () {

echob "Skipping ${PLATFORM} platform ........"
SF_STAT=NA ; SF_MSG=NA
}
#
#########
if ! [[ ${PLATFORM} == "KVM" || ${PLATFORM} == "VMware" ]] ; then
    chk_cmd lshw
    if [[ $(lshw -c net -quiet -businfo | grep SFC >/dev/null 2>&1 ; echo $?) == 0 ]] ; then
        SFC_CARD=$(lspci -vv |  grep SF | awk '/PN/ { print $4 }' | sort -u | paste -s -d " ")
        #echob "SFC card(s) ${SFC_CARD} installed on the host"
        if [[ -f /usr/sbin/sfupdate && -f /usr/bin/onload ]] ; then
          ONLOAD_CUR=$(/usr/bin/onload --version | awk '/EnterpriseOnload/ { print $2 }')
          for sfcard in ${SFC_CARD}
          do
            echob "SFC card ${sfcard} being checked "
            SF_OUT="$(/usr/sbin/sfupdate )"
            LSPCI_OUT="$(lspci -vv )"
            SYS_OUT="$(systool -c net)"
            SF_PCI=$(echo "${LSPCI_OUT}" | grep -B1 $sfcard | egrep -v "$sfcard|\--" | awk '{ print $1 }' | paste -s -d" ")
            SF_PORTS=$(find_sf_ports)

            SF_FW_REF="http://${REPO}/scripts/SF/SFC_FW.txt"
            wget --waitretry=5 --tries=2 --timeout=10 -q ${SF_FW_REF}
            if [[ $? -ne 0 ]] ; then
                echor "wget of ${SF_FW_REF} failed/not exist "
                SF_STAT=ERROR ; SF_MSG="wget"
            else
                set -- $(cat SFC_FW.txt | grep -v "#" | grep RHEL${OSREL} | grep ${sfcard})
                ONLOAD_EXP=$2 ; CNT_VER_EXP=$3
                echob "Expected  ONLOAD_VER: ${ONLOAD_EXP}  CNT_VER: ${CNT_VER_EXP} "
                SF_STAT=OK ; SF_MSG=OK
                for port in ${SF_PORTS}
                do
                    get_sfc_cur $port
                    cmp_sfc_fws $port
                    ONLOAD_CHK="done"
                done
                echo -e "\n\n"
                if [[ $sfcard == "SFN7x22F" ]] ; then chk_ethtool_opts ; fi
            fi
            echo -e "\n\n"
          done
        else
            echor "No SFUTIL/ONLOAD installed "
            SF_STAT=ERROR SF_MSG="NOUTILS"
        fi
    else
        if [[ ${LOWLAT} == "Y" ]] ; then
            echor "NO SFC card(s) on the host"
            SF_STAT=ERROR SF_MSG="ERROR"
        else
            echob "NO SFC card(s) on the host"
            SF_STAT=NA SF_MSG="NA"
        fi
    fi

else
    kvm_vmware_sfc_fw
fi
summary "SF Cards " "${SF_STAT}"
status_tag SF ${SF_STAT} ${SF_MSG}

}
# -------------------------------------------------------------------
CHK_ROUTES () {
# -------------------------------------------------------------------

chk_cmd /sbin/ip
ROUTES_STAT=OK
ROUTES_EXP=$(egrep -v "^#|^$" /etc/sysconfig/network-scripts/route-{bond{0..15},eth{0..15},br{0..15}} 2>/dev/null | awk -F: '{ print $NF }')
ROUTES_CUR="$(/sbin/ip route)"
AROUTES=$(ip ro |  egrep -v 'scope|default')
echo "--------- current routes ---------------"
echo "${ROUTES_CUR}"
echo "-----------------------------------------"
if ! [[ ${ROUTES_EXP} == "" ]] ; then
    ROUTES_CUR="$(/sbin/ip route)"
    while read route
    do
        echo "${ROUTES_CUR}" | grep -w "$route" > /dev/null 2>&1
        if [[ $? == 0 ]] ; then
            echo "$route exists - OK"
        else
            echor "$route NOT exist "
            ROUTES_STAT=ERROR
        fi
    done < <(echo "${ROUTES_EXP}" | sed 's/\/32//g')
else
    echo "No static routes configured - INFO"
    ROUTES_STAT=NA
fi
#
while read croute
do
echo "${ROUTES_EXP}" | sed 's/\/32//g' | grep "${croute}" >/dev/null
if [[ $? != 0 ]] ; then
  echor "${croute} NOT a configured static route "
  ROUTES_STAT=ERROR
fi
done < <(echo "${AROUTES}")

summary "STATIC Routes " "${ROUTES_STAT}"
#[ ! ${ROUTES_STAT} = ERROR ] && echog "STATIC Routes - ${ROUTES_STAT}"

status_tag ROUTES ${ROUTES_STAT} ${ROUTES_STAT}

}
# -------------------------------------------------------------------
CHK_MOUNTS () {
# -------------------------------------------------------------------


MNT_STAT=OK
ERR_MNTS=""
MOUNTS=$(egrep "ext|nfs" /etc/fstab | awk '!/^#/ { print $2 }')

for mnt in ${MOUNTS}
do
    if [[ $(grep -w $mnt /etc/mtab | grep -w rw >/dev/null 2>&1 ; echo $?) == 0 ]] ; then
        echo "$mnt: mount - OK"
    else
        echor "$mnt: mount "
        MNT_STAT=ERROR; ERR_MNTS="${ERR_MNTS} $mnt"
    fi
done

summary "FS mount " "${MNT_STAT}"

status_tag MOUNTS ${MNT_STAT} ${MNT_STAT}

}
# -------------------------------------------------------------------
CHK_SVCS () {
# -------------------------------------------------------------------


if [[ ${PLATFORM} == "HP" ]] ; then
    SVCS="snmpd hp-health collectl OVCtrl ovpa"
else
    SVCS="snmpd collectl OVCtrl ovpa"
fi
#
if [[ ${OSVER} ==  7 ]] ; then
  if [[ ${PLATFORM} == "HP" ]] ; then
    SYSCTL_SVCS="collectl hp-health snmpd"
  else
    SYSCTL_SVCS="collectl snmpd"
  fi
  LEGACY_SVCS="OVCtrl ovpa"
fi
#
if [[ $(lshw -c net -quiet -businfo | grep SFC >/dev/null 2>&1 ; echo $?) == 0 ]] ; then
   SVCS="${SVCS} sfc-rx-tune"
fi
#
if [[ ${LOWLAT} == "Y" ]] ; then
  if [[ ${OSVER} ==  6 ]] ; then
     SVCS="${SVCS} icap-lowlat"
  else
    SYSCTL_SVCS="${SYSCTL_SVCS} iebfisa-lowlat"
  fi
fi
SVCS_STAT=OK
#
#
newrh_svcs () {
 newsvc=$1

   if [[ $newsvc == "iebfisa-lowlat" ]] ; then
      systemctl status $newsvc | grep 'Active: active (exited)'
        if [[ $? != 0 ]] ; then
          echor "$newsvc NOT active - ERROR"
          SVCS_STAT=ERROR
        else
          echo "$newsvc is active - OK"
        fi
   else
      systemctl status $newsvc | grep 'Active: active (running)'
        if [[ $? != 0 ]] ; then
          echor "$newsvc NOT active"
          SVCS_STAT=ERROR
        else
          echo "$newsvc is active - OK"
        fi
   fi
}

legacy_svcs () {

  for svc in ${SVCS}
  do
    if [[ $(/sbin/chkconfig --list $svc 2>/dev/null| grep -qw on ; echo $?) == 0 ]] ; then
        echo "$svc added to chkconfig - OK"
        if [[ $(/sbin/service $svc status > /dev/null 2>&1; echo $?) == 0 ]] ; then
                echo "$svc is running  - OK"
        else
                echor "$svc service is NOT running "
                SVCS_STAT=ERROR
        fi

    else
        echor "$svc NOT added/ON in chkconfig "
        SVCS_STAT=ERROR

    fi

  done
}
##
if [[ ${OSVER} != 5 && ${OSVER} != 6 ]] ; then
  for svc in ${SYSCTL_SVCS}
  do
   newrh_svcs $svc
  done
  SVCS=${LEGACY_SVCS}
  legacy_svcs
else
  legacy_svcs
fi
#[ ! ${SVCS_STAT} == ERROR ] && echog "${SVCS} : service status "
summary "Services " "${SVCS_STAT}"

status_tag SVCS ${SVCS_STAT} ${SVCS_STAT}


}
# -------------------------------------------------------------------
CHK_RAID () {
# -------------------------------------------------------------------
HW_STAT=OK

raid_cfg () {

arr_cfg=$($hpscript controller all show config)
echo ${MODEL} | egrep "Gen8|Gen9" > /dev/null 2>&1
if [[ $? == 0 ]] ; then
        echo "$arr_cfg" | grep -w "Smart Array P440" | grep -w "Slot 0" > /dev/null 2>&1
        if [[ $? == 0 ]] ; then echor "Smart Array P440 exist on ${MODEL}" ;HW_STAT=ERROR; fi
        DTSPEED="$($hpscript ctrl slot=0 pd all show detail| grep Transfer | awk '{ print $4 }')"
        #echo ${MODEL} | egrep "Gen9" > /dev/null 2>&1
        #if [[ $? == 0 ]] ; then
        #   echo "${DTSPEED}" | grep "12.0Gbps"
        #   if [[ $? != 0 ]] ; then echor "${MODEL} DiskTransfrerSpeed not 12.0Gbps" ;HW_STAT=ERROR; fi
        #fi
fi

$hpscript ctrl slot=0 show config | grep 'logicaldrive 1' | grep 'RAID 0' > /dev/null
if [[ $? != 0 ]] ; then
  echoa "LD 1 is NOT a RAID 0 Array "
  echoa "Skipping further Boot Volume checks as no RAID 0 array "
else
  $hpscript ctrl slot=0 show config detail | egrep '[Primary|Secondary] Boot Volume' | grep None
  if [[ $? == 0 ]] ; then
      echor "Primary/Secondary Boot Volume not set properly "
      HW_STAT=ERROR
  fi
fi

if [[ $( echo "${arr_cfg}" | grep RAID | grep "RAID" | egrep -vc "OK|RAID 6 (ADG) Status" ) -gt 0 ]] ; then
    echor "Logical Drive/RAID status "
    HW_STAT=ERROR
else
    echo "Logical Drive/RAID status - OK"
fi
echo "${arr_cfg}"
if ! [[ $(echo "${OSREL} < 6.6 " | bc ) == 1 ]] ; then
    R0_LDS=$(echo "${arr_cfg}" | grep -c "RAID 0")
    if [[ $( echo "${R0_LDS}%2" | bc ) != 0 ]] ; then
        echob "Assuming EVEN number of RAID-0 LDs should exist on 6.6 and above"
        echor "incorrect RAID-0 logical drives "
        HW_STAT=ERROR
    fi
fi
}
#
ilo_mail () {

echo ${MODEL} | egrep "G7" > /dev/null 2>&1
if [[ $? != 0 ]] ; then
  chk_cmd /sbin/hponcfg
  /sbin/hponcfg -a -w ${STOR_DIR}/ilo.log
  grep 'ALERTMAIL_EMAIL_ADDRESS VALUE="BTEC.ALERTS@nex.com"' ${STOR_DIR}/ilo.log
  if [[ $? == 0 ]] ; then
    grep 'ALERTMAIL_SMTP_SERVER VALUE="10.44.167.200"' ${STOR_DIR}/ilo.log
    if [[ $? == 0 ]] ; then
      grep 'ALERTMAIL_ENABLE VALUE="Y"' ${STOR_DIR}/ilo.log
      if [[ $? == 0 ]] ; then
        echog "iLO alerts notification has been set "
      else
        echor "iLO alerts notification has NOT been set "
      fi
    else
      echor "iLO alerts notification has NOT been set "
    fi
  else
    echor "iLO alerts notification has NOT been set "
    HW_STAT=ERROR
  fi
else
  echob "Skipping iLO mail config check for ${MODEL} "
fi

}
#
disk_speed () {
echo ${MODEL} | egrep "Gen8|Gen9" > /dev/null 2>&1
if [[ $? == 0 ]] ; then
        $hpscript ctrl slot=0 pd all show detail | egrep 'Model|physicaldrive' | awk '{ print $NF }' | paste -s -d" \n" | while read line
        do
         set -- $line
        echo $line | grep 300 >/dev/null 2>&1
        if [[ $? == 0 ]] ; then
           TYPE=$(echo $2 | awk '{print substr ($0, 7, 1)}')
           if [[ $TYPE == J ]] ; then
             TYPE=12.0Gbps
           elif [[ $TYPE == F ]] ; then
             TYPE=6.0Gbps
           else
             TYPE="Undefined"
           fi

           if [[ $MODEL == *Gen9 ]] ; then
                  if [[ $TYPE == 12.0Gbps ]] ; then
                     echog "$line $TYPE OK"
                  else
                     echoa "$line $TYPE Drive model"
                  fi
           elif [[ $MODEL == *Gen8 ]] ; then
                  if [[ $TYPE == 6.0Gbps ]] ; then
                     echog "$line $TYPE OK"
                  else
                     echoa "$line $TYPE Drive model"
                  fi
           fi
        else
          echob "$line - skipping drive TX speed check"
        fi

        done
fi

}
#
md_devs=$(awk '/ md/ {print $NF }' /proc/partitions )
if [[ ${md_devs} != "" ]] ; then
    cat /proc/mdstat
    for md in `echo "${md_devs}"`
    do
        grep -A1 $md /proc/mdstat | grep 'UU' > /dev/null
        if [[ $? != 0 ]] ; then
            echor "$md : status "
            HW_STAT=ERROR
            cat /proc/mdstat
        fi
    done
fi
#
if [[ ${PLATFORM} == "HP" ]] ; then
    if [[ $(hplog -f | egrep -i 'fail|degra' >/dev/null 2>&1 ; echo $?) == 0 ]] ; then
        HW_STAT=ERROR
        echor "FAN failure "
        hplog -f
    else
      hplog -f | awk '{ print $7 }'  | grep No > /dev/null
      if [[ $? == 0  ]]  ; then
          echor "System Fans not Redundant "
          hplog -f
          HW_STAT=ERROR
      else
              echog "All FANS status "
        hplog -f
      fi
    fi

    if [[ $(hplog -p | egrep -i 'fail|degra' >/dev/null 2>&1 ; echo $?) == 0 ]] ; then
        HW_STAT=ERROR
        echor "PS failure "
        hplog -p
    else
        echog "All PSs status "
        hplog -p
    fi
    [ -f /usr/sbin/ssacli ] && hpscript=/usr/sbin/ssacli || hpscript=hpssacli
    chk_cmd $hpscript
    arr_cfg_detail=$($hpscript controller all show config detail | egrep -v "^Note:|Spare Activation")
    echo -e "------ Array config ----- \n${arr_cfg}"
    if [[ $(echo "${arr_cfg_detail}" | grep -ci fail) -gt 0 ]]; then
        echo "${arr_cfg_detail}" | grep -i fail
        echor "Internal disk/battery failure detected: "
        HW_STAT=ERROR
    elif [[ $(echo "${arr_cfg_detail}" | grep -c "Cache Board Present: True") -gt 0 ]] ; then
         if [[ $(echo "${arr_cfg_detail}" | grep -c "Cache Status: OK") -gt 0 ]] ; then
             echog "Cache Battery looks good "
         else
             CACHE_STAT=$(echo "${arr_cfg_detail}" | grep "Cache Status:" | awk -F ":" '{ print $2 }')
             echor "Cache battery ${CACHE_STAT}"
             echo "${arr_cfg_detail}"
             HW_STAT=ERROR
         fi
    else
        echog "All internal disks are looking good "
    fi
    raid_cfg
#   disk_speed
    ilo_mail
elif [[ ${PLATFORM} == "IBM" ]] ; then
    if [[ -f /opt/MegaRAID/MegaCli/MegaCli64 ]] ; then
        megacli=/opt/MegaRAID/MegaCli/MegaCli64
        pd_stat=$($megacli -PDlist -Aall | grep 'Firmware state:' | grep -i 'fail|error')
        ld_stat=$($megacli -LDInfo -Lall -aALL | grep -i degra)
        if [[ ${pd_stat} != "" || ${ld_stat} != "" ]] ; then
            HW_STAT=ERROR
            echor "physical disk or logical drive failures "
            $megacli -PDlist -Aall
            $megacli -LDInfo -Lall -aALL
        fi
    else
        chk_cmd /usr/local/bin/arcconf
        /usr/local/bin/arcconf GETCONFIG 1 | grep -i 'fail|error'
        if [[ $? == 0 ]] ; then
            HW_STAT=ERROR
            echor "physical disk failure/errors "
            /usr/local/bin/arcconf GETCONFIG 1
        fi
    fi
else
    echo "Skipping ${PLATFORM} platform "
    HW_STAT=NA
fi

summary "STORAGE_ARRAY/Battery/PS/FAN/iLO-Mail Checks " "${HW_STAT}"
#
status_tag RAID ${HW_STAT} ${HW_STAT}

}
# -------------------------------------------------------------------
CHK_SPACE () {
# -------------------------------------------------------------------

FS_STAT=OK
df_out=$(df -HP | grep -vE '^Filesystem|tmpfs|cdrom|rhel-dvd|\.iso')
df -HP
#
while read line
do
    set -- $line
    if [[ $(echo $5 | sed 's/%//g') -gt 89 ]] ; then
        echor "$6: file system usage is at risk($5) "
        FS_STAT=ERROR
    fi
done < <(echo "${df_out}")

summary "FileSystems usage " "${FS_STAT}"
#
status_tag FS ${FS_STAT} ${FS_STAT}


}

# -------------------------------------------------------------------
CHK_ILO () {
# -------------------------------------------------------------------


ilo_cfg () {

ILO_STAT=OK
dom_name_exp=${DOMAIN}
ilo_name_exp=$(hostname -s | sed 's/[pmcx]$/c/')
sshkeyfile=ilokey
if [[ ${SITE} == SECLAB || ${SITE} == SEC3 || ${SITE} == SLOLAB ]] ; then

sshkey_exp="0f:99:95:ae:20:62:e8:48:8a:63:5e:ff:fd:03:23:98:b8:a2:51:9e"
cat << EOF > ${sshkeyfile}
-----BEGIN DSA PRIVATE KEY-----
MIIBuwIBAAKBgQD8Lz4T7qVGm62kPSsAo0RS/ofpKQyqr+NEUDUaoSJjQM/oSqEV
7fUol4BvhCNhy/WXIxt4ANtfibeOy93ww5dGusMGLdmDGqGsibZTgb91fka691zD
E7lTg4H7syu2vu2lrYnWpSKYiosezBQfcrmz4wzmlU1qKvN+exDt6DtEzwIVAMl9
YuOvFMFbGE762VSHVtKO8YqhAoGBAOwcaboL2PZ2Ly55oTGD3wJoS99jIHgH14sm
flb5I/vztTP97OmtmVQu2fGlJ31F3fAJct2ZSFb4nx+TpSIighTEilhOTx8JBs3g
CaQp9SoqU4Qv2WzGb1mwfE3VMtxD2CRtIpXoZAnF266vyMPPt62y80TC3LLliuZL
5CGuzTY0AoGAZAmcurrn5f0tHNh7xt1Efq4NgizsvATK8F7LSL0+uOAmYrLdUR04
VGZbyRZ3IL/Fdgh/6T4/4oacmLSW6EYP3sAflSzJTuCBRoSG93oP7WreCzk8wOeK
Mbt6G7TFfztU3AHENla9rbGF1HPedeIe0Gz/ssnyu7FEFSJWkaSEZc8CFFGdYP49
H2AX5ILPSvbPw4N2V3bL
-----END DSA PRIVATE KEY-----
EOF

else

sshkey_exp="fe:95:fa:6d:f8:89:8b:e9:84:d5:17:8c:6f:c7:bb:f2:1c:34:54:b6"
cat << EOF > ${sshkeyfile}
-----BEGIN DSA PRIVATE KEY-----
MIIBuwIBAAKBgQCZpwxHIf/lOli4lVR/1Vm+xKT47CI/8KvccbQpPbAnZ/k/quES
t6TGwH61WU3Gp2aOBHwo6D7Fv8IbEJZxBomrJm6qQAaft1J1UmLCCNp/nYQIgZUx
TZeQUYEbVlJB17bwPao/t42lDDzI/MRYMxLofpno46ojXOAsjybQeK+0IwIVAMiW
D8MEmvQ2Q0MshhtyuYc23cGlAoGBAIsdxP1M/iRQj9ANL38Q9+FZyZWXmhEa7pMN
lL+F/EPfUpIgSV4VIQryfUuUOPAxdxm0w7+vZNh6YROQ0x4NuvUHE8wEI7bVvbmO
AxMooPQyHvkx6Fc00nBJDM/k+EK+YJW2S5I7uy8a6bFbBPGsLCVj1PBCbqTO4rbJ
lNSwwXvTAoGAVW81D9R56eg45MT+JtBzwSF2wgFyXZB3iqdODXtEOFMpYY4EjF6Z
6+w59H+epC+hoAskZg44sYhePUZEZXYhTtGhJhfOUCR3+Nird0MjgAEV4KH4vVKT
qH2fUggz031eiIg8WZSjjqyf4E9r+QZC7x/SOGZdswLXnkx48ypauj8CFBE4arqy
wo1aR4i2uKLTG7K9sfhO
-----END DSA PRIVATE KEY-----
EOF
fi
chmod 400 ${sshkeyfile}
####
#
#
host ${ilo_name_exp}  >/dev/null 2>&1
if [[  $? ==   0 ]] ; then
    echog "ILO ${ilo_name_exp} Name exist in DNS "
else

    ILO_STAT=ERROR ; echor "${ilo_name_exp} not in DNS database"
    ilo_name_exp=${ILOIP}
fi
chk_cmd /usr/bin/nc
/usr/bin/nc -z -w 5 ${ilo_name_exp} 22 >/dev/null 2>&1

if [[ $? == 0 ]] ; then
    [ -f /usr/bin/sshpass ] && ssh_pass_cmd=/usr/bin/sshpass || ssh_pass_cmd=/usr/local/bin/sshpass
    chk_cmd $ssh_pass_cmd
    ${ssh_pass_cmd} -p 'PASSW0RD' ssh -o 'StrictHostKeyChecking=no'  -o 'PubkeyAuthentication=no' Administrator@${ilo_name_exp} 'show /map1/accounts1/Administrator' | tr -dc '[:print:]\n' > ilo.out
    ${ssh_pass_cmd} -p 'PASSW0RD' ssh -o 'StrictHostKeyChecking=no'  -o 'PubkeyAuthentication=no' Administrator@${ilo_name_exp} 'show /map1/dnsendpt1' | tr -dc '[:print:]\n' >> ilo.out
    cat ilo.out
    sshkey_cur=$(awk -F"=" '/sshkey/ { print $2 }' ilo.out )
    ilo_name_cur=$(awk -F"=" '/Hostname/ { print $2 }' ilo.out )
    dom_name_cur=$(awk -F"=" '/DomainName/ { print $2 }' ilo.out )

    if [[ ${ilo_name_cur} == ${ilo_name_exp} && ${dom_name_cur} == ${DOMAIN} && ${sshkey_cur} == ${sshkey_exp} ]] ; then
        echog "current ILO name, Domain and ssh keys are as expected"
        echog "Current : ILO: ${ilo_name_cur}  DOMAIN: ${dom_name_cur}  SSHKEY: ${sshkey_cur}"
        echob "Expected: ILO: ${ilo_name_exp}  DOMAIN: ${DOMAIN}  SSHKEY: ${sshkey_exp}"
        ssh -i ${sshkeyfile} -l Administrator ${ilo_name_cur} power >/dev/null 2>&1
        if [[ $? == 0 ]] ; then
            echog "ILO ssh trusted access status "
        else
            echor "ILO ssh trusted access status "
        fi
    else
        ssh -i ${sshkeyfile} -l Administrator ${ilo_name_cur} power >/dev/null 2>&1
        if [[ $? == 0 ]] ; then
            echog "ILO ssh trusted access status "
        else
            echor "ILO ssh trusted access status "
        fi
        ILO_STAT=ERROR
        echor " Please verify listed CURRENT and EXPECTED settings"
        echor "Current : ILO: ${ilo_name_cur}  DOMAIN: ${dom_name_cur}  SSHKEY: ${sshkey_cur}"
        echob "Expected: ILO: ${ilo_name_exp}  DOMAIN: ${DOMAIN}  SSHKEY: ${sshkey_exp}"
    fi
else
    ILO_STAT=ERROR
    echor "ILO hostsystem $ilo_name_exp NOT reachable"
fi
}

#############
if [[ ${PLATFORM} == "HP" ]] ; then
    ilo_cfg
else
    echo "Skipping ${PLATFORM} platform "
    ILO_STAT=NA
fi

summary "ILO Name, Domain and sshkey " ${ILO_STAT}
status_tag ILO ${ILO_STAT} ${ILO_STAT}
echo "---------------------------------------------------------------------------------------------------------"
cd /tmp ; rm -rf /tmp/maintenance/${TS}
}

# -------------------------------------------------------------------
CHK_ORA12 () {
# -------------------------------------------------------------------

ORA12_STAT=OK

LIMITS="oracle soft nproc 2047
oracle hard nproc 16384
oracle soft nofile 1024
oracle hard nofile 65536
oracle soft stack 10240
oracle hard stack 10240"

SYSCTL="kernel.shmmni = 4096
kernel.shmmax = 53687091200
kernel.shmall = 4938529
fs.file-max = 6815744
net.core.rmem_default = 262144
net.core.wmem_default = 262144
fs.aio-max-nr = 1048576
kernel.sem = 250 32000 100 142
kernel.msgmni = 2878
kernel.msgmax = 65536
net.ipv4.tcp_syncookies = 1
kernel.core_uses_pid = 1
kernel.sysrq = 1
net.ipv4.ip_local_port_range = 9000 65535
kernel.msgmnb = 65536
kernel.panic_on_oops = 1"

rpm -q go-icus >/dev/null 2>&1
if [[ $? != 0 ]] ; then
  SYSCTLDB="
net.core.rmem_max = 4194304
net.core.rmem_max = 4194304"
else
  SYSCTLDB="
net.core.wmem_max = 134217728
net.core.rmem_max = 134217728"
fi

RPMPKGS="binutils
compat-libcap1
compat-libstdc++-33
cpp
gcc
gcc-c++
glibc
glibc-devel
glibc-headers
kernel-headers
ksh
libaio
libaio-devel
libdmx
libgcc
libstdc++
libstdc++-devel
libX11
libXau
libxcb
libXext
libXi
libXmu
xorg-x11-xauth
libXt
libXtst
libXv
libXxf86dga
libXxf86misc
libXxf86vm
make
mpfr
sysstat
xorg-x11-utils"

chk_ora_pkgs () {
chk_cmd rpm
for pkg in ${RPMPKGS}
do
    pkg_cur=$(rpm -qa $pkg)
    if [[ ${pkg_cur} == "" ]] ; then
        echor "$pkg: NOT installed "
        ORA12_STAT=ERROR
    else
        echo "$pkg_cur: installed - OK"
    fi
done

}

chk_ora_sysctl () {
chk_cmd /sbin/sysctl
sysctl_cur=$(/sbin/sysctl -a | sed 's/[[:space:]]//g')
while read line
do
    mline=$(echo $line | sed 's/[[:space:]]//g' )
    echo "${sysctl_cur}" | grep -w ${mline} >/dev/null 2>&1
    if [[ $? == 0 ]] ; then
        echo "$line: exists - OK"
    else
        echor "$line: NOT set "
        ORA12_STAT=ERROR
    fi
done < <(echo "${SYSCTL}" "${SYSCTLDB}")

}


chk_ora_limits () {
limits_cur=$(cat /etc/security/limits.conf | sed 's/[[:space:]]//g')
while read line
do
    mline=$(echo $line | sed 's/[[:space:]]//g' )
    echo "${limits_cur}" | grep -w "^${mline}" >/dev/null 2>&1
    if [[ $? == 0 ]] ; then
        echo "$line: exists - OK"
    else
        echor "$line: NOT set "
        ORA12_STAT=ERROR
    fi
done < <(echo "${LIMITS}")
}

chk_ora_usr_grp () {

id oracle | grep oinstall | grep dba >/dev/null 2>&1
if [[ $? == 0 ]] ; then
    echo -e "\n\n----------------"
    id oracle
    echo -e "----------------\n\n"
    if [[ $(ls -ld /u0[1-3] >/dev/null 2>&1; echo $?) != 0 ]] ; then
                if [[ $(ls -ld {/ora01,/data/{omex_[12],backups}} >/dev/null 2>&1; echo $?) != 0 ]] ; then
                        echor "Neither /u0[1-3] nor Genium db filesystems exist"
                        ORA12_STAT=ERROR
                else
                        ls -ld {/ora01,/data/{omex_[12],backups}}
                        ls -ld {/ora01,/data/{omex_[12],backups}} | grep -vw "oracle oinstall" >/dev/null 2>&1
                        if [[ $? == 1 ]] ; then
                                echog "oracle user id and {/ora01,/data/{omex_[12],backups}} file systems perms "
                        else
                                echor "Check oracle user id and {/ora01,/data/{omex_[12],backups}} file systems perms "
                                ORA12_STAT=ERROR
                        fi
                fi

    else
        ls -ld /u0[1-3]
        ls -ld /u0[1-3] | grep -vw "oracle oinstall" >/dev/null 2>&1
        if [[ $? == 1 ]] ; then
            echog "oracle user id and /u0[1-3] file systems perms "
        else
            echor "Check oracle user id and /u0[1-3] file systems perms "
            ORA12_STAT=ERROR
        fi
    fi
else
    echor "No oracle user id exist "
    ORA12_STAT=ERROR

fi

}

##############
grep "^APPLICATION:" ${BUILD_FILE} | grep DB >/dev/null 2>&1
if [[ $? == 0 ]] ; then
  id oracle >/dev/null 2>&1
  if [[ $? == 0 ]] ; then
    chk_ora_pkgs
    chk_ora_sysctl
    chk_ora_limits
    chk_ora_usr_grp
  else
    ORA12_STAT=NA
  fi
else
    ORA12_STAT=NA
    echob "Non DB host: Skipping ......."
fi

summary "Oracle12 DB " ${ORA12_STAT}
status_tag ORA12 ${ORA12_STAT} ${ORA12_STAT}

}

# -------------------------------------------------------------------
CHK_QLEN () {
# -------------------------------------------------------------------
qlen_cur=$(/sbin/sysctl -a | egrep  "net.ipv4.neigh.(default|(etn|eno|ens|eth|bond|br|)[0-9]).unres_qlen =")
echo "${qlen_cur}" | grep -vw 100 >/dev/null 2>&1
if [[ $? == 1 ]] ; then
    echog "QLEN settings are looking good "
    QLEN_STAT=OK
else
    echor "QLEN settings are NOT right "
    QLEN_STAT=ERROR
fi
echo "${qlen_cur}"
summary "Qlen settings " ${QLEN_STAT}
status_tag QLEN ${QLEN_STAT} ${QLEN_STAT}
}

# -------------------------------------------------------------------
CHK_LLTB () {
# -------------------------------------------------------------------


BIOS_REF='name="CPU_Virtualization:Disabled
name="Intel_Hyperthreading:Disabled
name="Intel_Processor_Turbo_Mode:Enabled
name="Intel_VT-d2:Disabled
name="Thermal_Configuration:Optimal_Cooling
name="HP_Power_Profile:Maximum_Performance
name="HP_Power_Regulator:HP_Static_High_Performance_Mode
name="Intel_QPI_Link_Power_Management:Disabled
name="Intel_Minimum_Processor_Idle_Power_State:No_C-States
name="Intel_Minimum_Processor_Idle_Power_Package_State:No_Package_State
name="Energy_Performance_Bias:Maximum_Performance
name="Energy_Performance_Bias_Gen9:Maximum_Performance
name="Collaborative_Power_Control:Disabled
name="Intel_DIMM_Voltage_Preference:Optimized_for_Performance
name="Dynamic_Power_Capping_Functionality:Disabled
name="Memory_Power_Savings_Mode:Maximum_Performance
name="QPI_Snoop_Configuration:Standard
name="System_Locality_Information_Table_G(7|8|9):Disabled
name="PowerMonitoring:Disabled
name="DisableMemoryPrefailureNotification:Yes
name="Memory_Refresh_(Gen8|Rate_Gen9):1x_Refresh'

if [[ ${PLATFORM} == HP ]] ; then
    LLTB_STAT=OK
[ -f /sbin/conrep ] && conrep_cmd=/sbin/conrep || conrep_cmd=/usr/sbin/conrep
  if [[  -f /usr/share/icap-hp-tools/conrep.xml && -f ${conrep_cmd} ]] ; then
    ${conrep_cmd} -s -f ./bios.out -x /usr/share/icap-hp-tools/conrep.xml >/dev/null
    echo "-------------------------------------------------------------------------------------------------------------------------"
    echob "$(printf "%-50s%-32s%-32s\n" "BIOS_PARAM:" "CUR_VALUE" "EXP_VALUE")"
    echo "-------------------------------------------------------------------------------------------------------------------------"
    while read i
    do
        param=$(echo "${i}" | cut -d ":" -f1)
        exp=$(echo "${i}" | cut -d ":" -f2-)
        cur=$(egrep -w "${param}" ./bios.out | sed -e "s/.*>\(.*\)<.*>/\1/g" )
        param=$(echo $param | sed 's/name="//g')
        if [[ "$cur" == "$exp" ]] ; then
            echog "$(printf "%-50s%-32s%-32s\n" "${param}:" "${cur}" "${exp}")"
        elif [[ "$cur" == "" ]] ; then
            echob "$param: ------------------------------ NOT exist --------------------------------------"
        else
            echor "$(printf "%-50s%-32s%-32s\n" "${param}:" "${cur}" "${exp}")"
            LLTB_STAT=ERROR
        fi

    done < <( echo "${BIOS_REF}" | grep -v "^#" )
    echo "-------------------------------------------------------------------------------------------------------------------------"
  else
    echor "conrep or conrep.xml not exist"
    LLTB_STAT=ERROR
  fi


else
    echob "Skipping ${PLATFORM} platform ..........."
    LLTB_STAT=NA
fi
summary "HP_LLT_BIOS Settings " ${LLTB_STAT}
status_tag LLTB ${LLTB_STAT} ${LLTB_STAT}
}
# -------------------------------------------------------------------
CHK_BUILD_FILE () {
# -------------------------------------------------------------------
BUILD_FILE_STAT=OK
if [[ -f ${BUILD_FILE} ]] ; then
    while read line
    do
        DATA=$(echo $line | awk -F":" '{ print $2 }')
        if [[ ${DATA} == "" ]] ; then
            set -- $line
            [ -z $1 ] && echoa "blank line or no data in the line" || echor "$1 is missing the data"
            #echor "$1 is missing the data"
            BUILD_FILE_STAT=ERROR
        else
            echo $line
        fi

    done < ${BUILD_FILE}
    if [[ $(awk '/BUILD VERSION:/ { print $3 }' ${BUILD_FILE} ) != $OSREL ]] ; then
        echor "${BUILD_FILE} OS ver not matching with installed ver "
        BUILD_FILE_STAT=ERROR
    fi
    if [[ ${PLATFORM} != "VMware" ]] ; then
        CAGE=$(grep CAGE ${BUILD_FILE} | awk '{ print $2 }')
        CABINET=$(grep CABINET ${BUILD_FILE} | awk '{ print $2 }')
        if [[ $CAGE == 'N/A' || $CABINET = 'N/A' ]] ; then
           echor "CAGE/CABINET info not set properly "
        fi
    fi
else

    echor "${BUILD_FILE} file is missing"
    BUILD_FILE_STAT=ERROR
fi

summary "${BUILD_FILE} file status " ${BUILD_FILE_STAT}
status_tag BUILD_FILE_STAT ${BUILD_FILE_STAT} ${BUILD_FILE_STAT}
}
# -------------------------------------------------------------------
CHK_VMTOOL () {
# -------------------------------------------------------------------

if [[ ${PLATFORM} == "VMware" ]] ; then
    /usr/bin/vmware-toolbox-cmd -v
    if [[ $? == 0 ]] ; then
        VMT_VER=$(/usr/bin/vmware-toolbox-cmd -v | awk '{ print $1 }')
        #if [[ ${VMT_VER} == "10.0.0.50046" ]] ; then
                VMTOOL_STAT=OK
        echog "VMtools Ver ${VMT_VER} is installed "
        #else
        #       echor "VMtools Ver ${VMT_VER} not matching with 10.0.0.50046 "
        #       VMTOOL_STAT=ERROR
        #fi
    else
        echor "VMtools error "
        VMTOOL_STAT=ERROR
    fi
else
    VMTOOL_STAT=NA
fi
summary "VMTOOLS Install status " ${VMTOOL_STAT}
status_tag VMTOOL ${VMTOOL_STAT} ${VMTOOL_STAT}
}
# -------------------------------------------------------------------
CHK_YUM_UPD () {
# -------------------------------------------------------------------

run_yum_chks () {
yum clean all
if [[ $(yum --disableplugin=*  check-update -x *-iceu -x enterprise* | egrep -cv "^$|^Excluding|^Finished|^http|Trying") != 0 ]] ; then
    echoa "yum update is NOT clean "
    yum --disableplugin=*  check-update -x *-iceu -x enterprise* | grep -v "^$"
    YUMUPD_STAT=WARN
else
    YUMUPD_STAT=OK
fi
}

if [[ ${REPO_STAT} == "OK" ]] ; then
    run_yum_chks
else
    echor "Yum REPO server ${REPO} NOT reachable "
fi

summary "yum update status " ${YUMUPD_STAT}
status_tag YUMUPD ${YUMUPD_STAT} ${YUMUPD_STAT}
}
# -------------------------------------------------------------------
CHK_DNS () {
# -------------------------------------------------------------------
cat /etc/resolv.conf
DNS_STAT=OK
#
resolv_conf () {
while read line
do
    grep -w "^$line" /etc/resolv.conf >/dev/null 2>&1
    if [[ $? != 0 ]] ; then
        echor "$line - missing in /etc/resolv/conf"
        DNS_STAT=ERROR
    fi
done < <(echo "${RSLVCONF}")

}
#
dns_records () {
IPS=$(/sbin/ip ad | grep -w inet | awk  '!/127.0.0|192.168|169.254.95/ { print $2 }' | awk -F"/" '{ print $1 }')
for inetip in ${IPS}
do
 dnsout=$(host $inetip)
 if [[ $? == 0 ]] ; then
    dname=$(echo ${dnsout} | awk '{ print $NF }' | cut -d"." -f1)
    dnsout1=$(host $dname)
    if [[ $? != 0 ]] ; then
        DNS_STAT=ERROR
        echor "$dname not exist in DNS "
    fi
 else
    DNS_STAT=ERROR
    echor "$inetip not exist in DNS "
 fi

done
}
#
chk_tmp_puppet_file () {

grep "TEMP FILE" /etc/hosts
if [[ $? == 0 ]] ; then
   echor "Puppet TEMP file contents exist "
   DNS_STAT=ERROR
fi
}
#
resolv_conf
dns_records
chk_tmp_puppet_file
summary "DNS Checks status " ${DNS_STAT}
status_tag DNS ${DNS_STAT} ${DNS_STAT}

}
# -------------------------------------------------------------------
# -------------------------------------------------------------------
# -------------------------------------------------------------------
# -------------------------------------------------------------------
################  run individual checks ###############

create_dump_dir
echob "----------------------------------------------------------------------------------------------------------------------------------------"
echo  "==>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>       `hostname -s`    <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<=="
echob "----------------------------------------------------------------------------------------------------------------------------------------"
for CHK in ${PARAM}
do
 top_b ${CHK}
 ${CHK}
 bot_b ${CHK}
done

#
cd /tmp ; rm -rf /tmp/maintenance/${TS}
echo "---------------------------------------------------END of Checks on `hostname -s` -------------------------------------------------------"
