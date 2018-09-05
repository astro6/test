
mkdir -p DL360Gen9/tmp/maintenance
copy after_install into DL360Gen9/tmp/maintenance
cd DL360Gen9/tmp/maintenance
rpm2cpio ../../hp-firmware-ilo4-2.55-1.1.i386.rpm | cpio -idumv
extract all three downloaded ilo/bios/ssa/ rpms
cd ../..

/opt/puppet/bin/fpm -s dir -t rpm -C iebfisa-tools --name iebfisa-tools -v 1.2 --iteration 0.0.el7 --description "iebfisa-tools 1.2" --url http://www.nex.com --license "none"  .

/opt/puppet/bin/fpm -s dir -t rpm -C DL360Gen9 --name DL360Gen9 --iteration 2.0.el7 --description "FW upd BIOS(01/22/2018), iLO4(2.55) and SSA(6.30)" --url http://www.nex.com --license "none" --after-install=DL360Gen9/tmp/maintenance/after_install.sh .








LEgacy Master
-------------

1.Log into current/legacy master - stop service for pe-console-services, pe-puppetdb, pe-puppetserver
	for svc in puppet pe-puppetserver pe-puppetdb pe-console-services pe-nginx pe-activemq pe-orchestration-services pxp-agent; do echo "Stopping $svc" ; puppet resource service $svc ensure=stopped; done
	for svc in puppet pe-puppetserver pe-puppetdb pe-console-services pe-nginx pe-activemq pe-orchestration-services pxp-agent; do echo "Starting $svc" ; puppet resource service $svc ensure=running; done

2. cd /bkup; mkdir cutover;cd cutover
   tar -zcvf $BACKUP_DIR/puppet_ssl.tar.gz /etc/puppetlabs/puppet/ssl/
	
3.rsync /bkup to new master /bkup 

4.reconfigure Legacy master with temp IPs

5. scp /root/.ssh/* to NEW Master

NEW MASTER
-------------
5.on new master, configure final IP, make sure DNS lookup work for hostname forward/reverse ,

6.update pe.conf file with admin password "Welcome1" and the strings at the bottom of proper puppet source dir 

7. restore ssl keys
   mkdir -p /etc/puppetlabs/puppet
   restore the SSL directory: tar -zxvf $BACKUP_DIR/puppet_ssl.tar.gz -C /


8. now run installer after changing to folder puppet-enterprise-2017.3.2-el-6-x86_64> # ./puppet-enterprise-installer 
	choose text mode
	
9.follow on screen instructions to a successful installation 

10. once the web console works , import the data as follows,
  1. become user pe-postgres ( changed shell to bash in passwd file temporarily)
  2. cd /bkup/daily<respective_date>/
  2. run /opt/puppet/bin/pg_restore -c -d "pe-rbac" -v rbac_* (rback file copied on step#2)
  3. run /opt/puppet/bin/pg_restore -c -d "pe-classifier" -v classifier_* (classifier file copied on step#2)
  
  
  for svc in puppet pe-puppetserver pe-puppetdb pe-console-services pe-nginx pe-activemq pe-orchestration-services pxp-agent; do echo "Stopping $svc" ; puppet resource service $svc ensure=stopped; done
  copy license.key, hiera.yaml and puppet.conf and /usr/local/bin/gitpull.sh from Legacy /etc/puppetlabs/ into NEW Master
  for svc in puppet pe-puppetserver pe-puppetdb pe-console-services pe-nginx pe-activemq pe-orchestration-services pxp-agent; do echo "Starting $svc" ; puppet resource service $svc ensure=running; done

11. Login to new web console with your old id/pass and make sure classification data exist as expected


12. Create Git pull mechanism for respective git repos
     and daily dump cron jobs on new master and disable legacy jobs on legacy master

labs
------
cd /etc/puppetlabs/code/
mkdir hieradata
chown pe-puppet:pe-puppet hieradata
copy /root/.ssh/* from current box

git clone secgit:puppet/newlab-puppet-modules.git modules/
git clone secgit:puppet/newlab-puppet-hieradata.git hieradata/

chown -R pe-puppet:pe-puppet hieradata/
chown -R pe-puppet:pe-puppet modules/

PROD:
------
git clone git@gitmaster-us-prod:puppet/newprod-puppet-modules.git modules
git clone git@gitmaster-us-prod:puppet/newprod-puppet-hieradata.git hieradata
chown -R pe-puppet:pe-puppet modules hieradata 
hown -R pe-puppet:pe-puppet modules/

pre-prod
--------
git clone git@secgit:puppet/newprod-puppet-modules.git modules
it clone git@secgit:puppet/newprod-puppet-modules.git hieradata

chown -R pe-puppet:pe-puppet modules hieradata 
hown -R pe-puppet:pe-puppet modules


run noop run master and overlay.

run noop run on all clients and make sure no errors.

=============================================================================================================================


sed "s/^#Role/profiles::base::ptp: 'yes'\n&/" sec1gbk01p.yaml







https://support.puppet.com/hc/en-us/articles/115004904627-KB-0107-Migrate-to-a-monolithic-installation-of-Puppet-Enterprise-2016-4-x-to-2017-3-x


1. current_master		# puppet cert clean <NEW MASTER CERTNAME>

2. export BACKUP_DIR=/bkup/cutover

3. mkdir $BACKUP_DIR

4. tar -zcvf $BACKUP_DIR/puppet_ssl.tar.gz /etc/puppetlabs/puppet/ssl/

5. Transfer the resulting tarball to the /bkup/cutover directory on the new master using your preferred method.

On the PuppetDB node, back up the databases:

1.	chown pe-postgres:pe-postgres $BACKUP_DIR

2. cd $BACKUP_DIR

3. for db in pe-activity pe-classifier pe-orchestrator pe-puppetdb pe-rbac; do echo "Backing up $db" ; sudo -u pe-postgres /opt/puppetlabs/server/bin/pg_dump -Fc $db -f $db.backup.bin; done

4. Transfer the resulting files to the /bkup/cutover directory on the new master using your preferred method.


NEW MATSR

Restore the SSL directory on NEW MASTER

	1.Set tmp/backup as the backup directory: export BACKUP_DIR=/bkup/cutover


	2.Create the directory: mkdir -p /etc/puppetlabs/puppet


	3.Restore the SSL directory: tar -zxvf $BACKUP_DIR/puppet_ssl.tar.gz -C /


--------------
Install PE  - 2017.x.x
-------------

Restore the databases and classifier data


1. export BACKUP_DIR=/bkup/cutover

2. cd $BACKUP_DIR

3. stop all services except for pe-postgresql

	for svc in puppet pe-puppetserver pe-puppetdb pe-console-services pe-nginx pe-activemq pe-orchestration-services pxp-agent; do echo "Stopping $svc" ; puppet resource service $svc ensure=stopped; done

4. Restore the databases from the backup directory:
/opt/puppetlabs/server/bin/pg_restore -c -d "pe-rbac" -v rbac_*
/opt/puppetlabs/server/bin/pg_restore -c -d "pe-classifier" -v classifier_*
for svc in puppet pe-puppetserver pe-puppetdb pe-console-services pe-nginx pe-activemq pe-orchestration-services pxp-agent; do echo "Stopping $svc" ; puppet resource service $svc ensure=stopped; done
for svc in puppet pe-puppetserver pe-puppetdb pe-console-services pe-nginx pe-activemq pe-orchestration-services pxp-agent; do echo "Starting $svc" ; puppet resource service $svc ensure=running; done


		for db in pe-activity pe-classifier pe-orchestrator pe-puppetdb pe-rbac; do echo "Restoring $db" ; /opt/puppetlabs/server/bin/pg_restore -Cc $BACKUP_DIR/$db.backup.bin -d template1; done
5. fix database permissions and recreate database extensions

	/opt/puppetlabs/bin/puppet-infrastructure configure

Deactivate and clear certificates on your old infrastructure nodes
	1.On the new master, run the following command:

	puppet node purge <OLD MASTER CERTNAME>; puppet cert clean <OLD MASTER CERTNAME>

Migrate configuration files, Puppet code, and Hiera data

   
	•Edit puppet.conf to add customizations from your old deployment on the new master.


	•If you use Code Manager or r10k, configure Code Manager or r10k to deploy code on the new master.


	•If you don't use Code Manager or r10k, copy the contents of the code directory /etc/puppetlabs/code/ to the new master.


	•Move your Hiera data and copy your old hiera.yaml file to /etc/puppetlabs/puppet/hiera.yaml on the new master.


	•Copy classification customizations to the new installation


		----------------------------------- ------------------------------------------------------
    -----------------------------------------------------------------------------------------
    
    install old-client curl -k https://puppet-us.if.icap.com:8140/packages/current/install.bash
run overlay
download zip
tarx xzvf
cd /data/downloads/puppet-enterprise-2017.1.1-el-6-x86_64
edit conf.d/pe.conf
for admin password and pe_install::puppet_master_dnsaltnames
[ "puppet-us-dr", "puppet-us-dr.if.icap.com", "puppet-us", "puppet-us.if.icap.com", "sti2pup01m", "sti2pup01m.if.icap.com", "sti2pup01p.if.icap.com" ]
update /etc/hosts
#
172.31.yy.xx sti2pup01p sti2pup01p.if.icap.com ( for m interface )

./puppet-enterprise-installer -c /data/downloads/puppet-enterprise-2017.1.1-el-6-x86_64/conf.d/pe.conf
( in case of uninstall
./puppet-enterprise-uninstaller -p -y -d )
login to pe console
reset admin password
create users
assign admin roles
make git setup

PROD:
git clone git@gitmaster-us-prod:puppet/newprod-puppet-modules.git modules
git clone git@gitmaster-us-prod:puppet/newprod-puppet-hieradata.git hieradata
chown -R pe-puppet:pe-puppet modules hieradata 


pre-prod

git clone git@secgit:puppet/newprod-puppet-modules.git modules

labs

cd /etc/puppetlabs/code/
mkdir hieradata
chown pe-puppet:pe-puppet hieradata
copy /root/.ssh/* from current box

git clone secgit:puppet/newlab-puppet-modules.git modules/
git clone secgit:puppet/newlab-puppet-hieradata.git hieradata/

chown -R pe-puppet:pe-puppet hieradata/
chown -R pe-puppet:pe-puppet modules/




=======================================================================================
puppet-2 notes
---------------------


puppet.conf settings - command line params after bash

- Pe Master modules
    pe_repo class
      complie_master_pool_address: value=hostname that you want on install.bash


**
    baseurl  => "http://localrepo-${region}-${env}/rhel-dvd-${operatingsystemrelease}/Server",
#   baseurl  => "http://localrepo-${region}-${env}/rhel-dvd-${::os[release][major]}/Server",

better yet:

#   baseurl  => "http://localrepo-${::region}-${::env}/rhel-dvd-${::os[release][major]}/Server",


puppet-lint  - rubygem executable
puppet parser validate

/opt/puppetlabs/puppet/bin/gem list


git pre-commit hook
git pre-receive hook


gem install puppet-lint --no-ri --no-doc ( skips docs install )


puppet-lint -f  ( updates files )

bash git prompt

file ensure ---

use ensure => 'file|directory|link|absent' instead 'present'


git hooks  in pre-commit config
puppet-lint ${file} & puppet parser validate  ${file}


~/.gitconfig
email
name
template_dir=<path_to_folder>


natemccurdy dotfiles and git-hubconfig

.vimrc  to auto fix puppet-lint issues

~/modules
.gitignore

foo.txt
foo/


sshd_config changes
package => service => file  sequence is required
notify required in both sshd_config sections

dmz servers daily checks ??


two soft tabs issue
   go to the line press 'shift + v' and use arrow to select whole block  and press '='
this fixes the aligning


array format with multiple elements
 
source => [
  'one',
  'two',
]


Ctrl+v on the column to the left + select lines and shift + i + perform action you want
+esc+esc - does same actions for all lines choosen


case statements
 needs default value too


snmp custom module - rhel version specific code

yum_repo_setup - fixed the hostconfig to profiles path


base.pp - split based on function and include in base.pp


day3

git commit -anm " to skip hooks checks"
master dump jobs
memory settings

ssl trust for newly onboarded servers?

alternate names setup review


SSL on master contains 3 types

	CA
	MASTERt agent
	All agents
	generate keys for dr
copy all these four into DR host

puppet infra configure  - will rerun the installer to fix any issues


subgroups creation issue

in variables sections keep defauts in the last

secondary groups creation issue


File['home/imon'] {
  group => 'data_tools',
}


/etc/puppetlabs/code/environments/production/manifests/site.pp


variables types
	global
	environment
	module

environment groups
	productions
	lab
	etc...
    no classes in these groups, only env setup

nex roles
    include classes


puppet config print modulepath ( prints current config path in use )


git log
git show


set list
set list!

base split


insert text in the first line of a file
sed -i "1s/^/profile::base:$i\n/" $i;


node_manager on forge by whatsaranjit



git reset HEAD <folder|file> to undo addition

classifier information
----------------------
whatsAranjit  node_manager forge module to be installed

puppet resource node_group <node_group_name>

remove id info before importing

modify the output

puppet apply output ( in destination )

puppet apply ( with ensure => absent ) will delete the node_group



puppet_db  endpoints
-------------

node info
fatcs
resources
reports

pe-client-tools installed on all masters by default


PQL - puppet query Language

puppet query 'nodes[certname,report_timestamp] { certname ~ "hbr3" }'

nodes|facts|inventory|resources|reports

inventory contains everything

puppet query ' resources[certname] { type = "User" and title = "ssystems" }

puppet acess login for token


remote puppet run , client needs px-agent running
-----------------
#puppet job run -q '  '

 and origin host needs pe-client-tools master has it installed by default

https://puppet.com/docs/puppetdb/5.1/api/query/v4/pql.html
https://puppet.com/docs/puppetdb/5.1/api/query/tutorial-pql.html
puppet query 'resources[certname] { }'  - gives host list
puppet query 'resources[certname] { type = "User" and title = "vvanka" }' - specific user
puppet query 'resources[title] { type = "User"}'  - lists of all users
puppet query 'resources[title] { type = "User"}' | grep title | sort -u

  "tags" : [ "roles::infra_e0_deployment", "class", "infra_e0_deployment", "user", "sa_user2", "profiles", "users", "vvanka", "profiles::users::sa_user2", "create_users2", "default", "roles", "node" ],
  "file" : "/etc/puppetlabs/code/modules/create_users2/manifests/init.pp",
  "type" : "User",
  "title" : "vvanka",
  "line" : 16,
  "resource" : "04e980b42879a78534ba474b6f5b91d96c716aa9",
  "environment" : "production",
  "certname" : "e0vingjm02p.if.icap.com",
  "parameters" : {
    "ensure" : "present",
    "uid" : "1508",
    "password_max_age" : "99999",
    "shell" : "/bin/bash",
    "managehome" : true,
    "home" : "/home/vvanka",
    "comment" : "Venkateswara Rao Vanka",
    "groups" : "wheel",
    "gid" : "1508"
  },
  "exported" : false


puppet query 'resources[certname] { tags = "roles::infra_e0_deployment" }' | sort -u ( roles by hosts )
puppet query 'resources { certname = "u0pengks01p.if.icap.com" }'

exclluding results
  type = "User" and
  title = "nick" and
  !(file = "/etc/puppetlabs/code/environments/production/manifests/user.pp" and line = 111)
}

facts endpoint:
=====================
to get h/w model:
puppet query 'facts[value] { certname = "u0pengks01p.if.icap.com" and name = "productname" }'

name" : "bios_release_date",


facts {
  name = "uptime_seconds" and
  value >= 100000 and
  value < 1000000
}










git add -A  ( will include all additions and deletions )


git checkout <file_name> - to revert back the changes


git reset --soft/hard



last enforce run
--------------------
puppet query 'reports[certname,end_time] { noop = false and certname ~ "u0p" }'


git pull --rebase && git push


visualizing git concepts
graphical view



120417
============

facter -p
eyaml - encrypt and decrypt

hiera 5

use syntax .yaml in hieradata paths



/etc/puppetlabs/facter/facts.d/buildinfo.yaml/json/txt

become facters

facter cage/lowlat


overlay run with puppet changes built in node groups only like mcollective

--tags  puppet-enterprise 

to install only plugins

puppet plugin download, it will happen with noop run too

puppet agent --noop --tags profiles::base:etc_hostconfig



razorsedge - network  module from forget




puppet enterprise support  ( like sosreport )





12062017
=========

code manager   service
  production
  foo

puppet code deploy ( can be run from anywhere )
  deploys into /etc/puppetlabs/code/environments/production(foo)



----

puppet-repo/
 hieradate/
 hiera.yaml
 manifests/
   site.pp
 modules/
   roles/
   profiles/


site.pp and hierayaml can be at differnt places
-----------------------

to refresh classes info in pup master git

curl -X POST https://$(hostname -f):4433/classifier-api/v1/update-classes?environment=production \
  --cert $(puppet config print hostcert) \
  --key $(puppet config print hostprivkey) \
  --cacert $(puppet config print cacert)




#
puppet upgrade
set git localtime
session timeout

puppet config print | grep -i class


agent -t --environment "test_base" --noop


git GUI

GitLab
BitBucket


git log
git show
git history


================

git pull
git checkout -b new_profile ( create a branch new_profile and changes to it )

anything you do here it doesn't affect anyone else

can be run from anywhere from repo


it doesn't create any folder but logical branch


git checkout master to go back to master


you should never making any commits in the master


create and test in branches and merge into master


git merge new_branch from master

git diff master new_branch


git revert<head>

git branch -d <new_branch>  ( deletes  the branch only locally )


between local and master

git push origin new_branch


on git master

git fetch    for tracking
git branch -a


.bashgit tool



zsh  fancy 




=========================================================================================

