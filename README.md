
#!/bin/bash

# venkateswara.vanka@nex.com - 08/17/2018
# rsync selective /home folders of selective hosts into jump host dump dir
# for rhel7 migrations

hlist=$1
#hlist=hosts-verified.txt
exlst=excludes.txt
dump_dir=/home_backup
rsync_opts="--progress -avzpPrlHh --delete-after --delay-updates --links --max-delete=1000 "
excludes=" --exclude /home/lost+found "
LOG=/home_backup/reposync.log
DATE=`date +%x-%R:%S`


###########################################################################
chk_dst_folder () {

ls -ld /${dump_dir}/$1 > /dev/null 2>&1 || mkdir -p /${dump_dir}/$1

}

gen_exlst () {

 for folder in $skips
 do
   excludes="$excludes --exclude /home/$folder "
 done
}

########
for hh in `cat $hlist | grep -v '#'`
do
 uhh=$(echo $hh | sed "s/.$//")
 grep $uhh ${exlst} > /dev/null 2>&1
 if [[ $? != 0 ]]; then
   chk_dst_folder $hh
   echo "---------------------- RSYNC of $hh at $DATE ----------------------" | tee -a $LOG
   rsync -e "ssh -i /root/.ssh/m2_lab_id_dsa -o StrictHostKeyChecking=no" $rsync_opts $excludes $hh:/home /${dump_dir}/$hh/ | tee -a $LOG
 else
   skips=$(grep $uhh ${exlst} | awk -F':' '{ print $2 }')
   if [[ $skips = "" ]] ; then
     continue
   else
     gen_exlst
     chk_dst_folder $hh
     echo "---------------------- RSYNC of $hh at $DATE ----------------------"  | tee -a $LOG
     rsync -e "ssh -i /root/.ssh/m2_lab_id_dsa -o StrictHostKeyChecking=no" $rsync_opts $excludes $hh:/home ${dump_dir}/$hh/ | tee -a $LOG
   fi
 fi
done
