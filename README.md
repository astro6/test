

  #!/bin/bash

display_usage() {
echo '====================================================='
echo "Usage: $0 [folder of collectl source files]"
echo '-----------------------------------------------------'
echo "Example: $0 /tmp/stage"
echo " "

    }

if [  $# -le 0 ] || [  $# -lt 1 ] ||[[ ( $# == "--help") ||  $# == "-h" ]]
    then
        display_usage
        exit 1
fi

for i in `ls $1/*.raw.gz`
do
 echo " --------- processing  $i ---------";
 collectl -p $i -P -f /data/collectl/plots -w -ocz -s+CND
 mv ${i} ${i}.done
 echo "----------------------------------------------------------"
done



