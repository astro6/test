

no git server and puppet master on same host
packages
puppet-agent   puppet facter hiera ruby openssl .........
puppet config print server/certname/environment
puppet parser validate review
puppet-lint ( test code )
puppet cert list ( pending )
				--all
puppet cert sign
RBAC module on master 
puppet resource rbac_user - shows currently configured users/roles
rbac_roles

git blame - gives entire history

git add -A ( deleted files )

gitweb/gitlab/bitbucket

node_manager to create node groups

plugin sync before facts transfer

facter os.release.major

facter -p

facter networking.interfaces.eth0.mac

# puppet config print resourcefile
/opt/puppetlabs/puppet/cache/state/resources.txt

shows type and title of resouce based on last catalog

# puppet describe --list

puppet parser validate review/
puppet epp validate review/template/motd.epp
'
puppet apply review/examples/init.pp --noop

pupply apply -e "include review" --noop

"template/notice" is a function in puppet 

classification - site.pp under environment/manifests/
this doesn't show in PE console

environment group and classification group

tree $(puppet agent --configprint resourcefile/clientcatadir/stderr)

util "jq"  jason query

puppet agent --configprint lastrunreport

for defaults , use "capital" letter in resourcetype
File
Exec
don't use resource defaults if you r including of other classes

TP scope r - console variables

all facts live top scope

local scope is local to the class 
--------------------------------
doc access code 0932
--------------------------------

# puppet module install puppetlabs/mysql
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ..

puppet describe mysql_user -s 

value of any string(with quotes ) in ppuppet is TRUE


puppet access login
puppet query "resources[]certname, title ] 
pe-client tools - set of addon to puppet

custom facts gets synced before sending facts


facter -p  to see inlcuding custom facts

external facts r placed on the client under /etc/.....facts.d/

json format, yaml format, and text format ( allows only strings)

root@vv:~/puppetcode/modules # mkdir -p /etc/puppetlabs/facter/facts.d
root@vv:~/puppetcode/modules # vim /etc/puppetlabs/facter/facts.d/test.yaml
root@vv:~/puppetcode/modules #
root@vv:~/puppetcode/modules #
root@vv:~/puppetcode/modules #
root@vv:~/puppetcode/modules # facter test
Hello world
root@vv:~/puppetcode/modules # facter test2
[
  "one",
  "two",
  "three"
]

 RUBYLIB="$PWD/kerberos/lib" facter default_realm

 file_line  type 
 
 ensure
 path
 line
 match
 
 AUgeas - 3rd party tools
 # /opt/puppetlabs/puppet/bin/augtool
augtool> ls /files/etc/yum.conf
main/ = (none)
augtool> ls /files/etc/yum.conf/main
cachedir = /var/cache/yum/$basearch/$releasever
keepcache = 0
debuglevel = 2
logfile = /var/log/yum.log
exactarch = 1
obsoletes = 1
gpgcheck = 1
plugins = 1
installonly_limit = 5
bugtracker_url = http://bugs.centos.org/set_project.php?project_id=19&ref=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg = centos-release
#comment[1] = This is the default, if you make this bigger yum won't see if the metadata
#comment[2] = is newer on the remote and so you'll "gain" the bandwidth of not having to
#comment[3] = download the new metadata and "pay" for it by yum not having correct
#comment[4] = information.
#comment[5] = It is esp. important, to have correct metadata, for distributions like
#comment[6] = Fedora which don't keep old packages around. If you don't like this checking
#comment[7] = interupting your command line usage, it's much better to have something
#comment[8] = manually check the metadata once an hour (yum-updatesd will do this).
#comment[9] = metadata_expire=90m
#comment[10] = PUT YOUR REPOS HERE OR IN separate files named file.repo
#comment[11] = in /etc/yum.repos.d
augtool>

augtool> get /files/etc/yum.conf/main/keepcache
/files/etc/yum.conf/main/keepcache = 0
augtool> set /files/etc/yum.conf/main/keepcache 1
augtool> get /files/etc/yum.conf/main/keepcache
/files/etc/yum.conf/main/keepcache = 1
augtool>


# cat /etc/puppetlabs/puppet/hiera.yaml
---
:backends:
  - yaml

:yaml:
  :datadir: /etc/puppetlabs/hieradata

:hierarchy:
   - host/%{site}/%{hostname}
   - app/%{app}
   - site/%{site}/%{app}
   - site/%{site}
   - global

   
  from hiera first match it takes
  unless hiera -a  
  
  classification - roles
  implementation - profiles
  resolves - modules
  
  must be one class per manifest
  
  for include init.pp is required

  puppet parser validate  - syntax checks 
  puppet-lint for style checks
  
  rspec-puppet
  
  proook

Pro  Puppet book 


  


