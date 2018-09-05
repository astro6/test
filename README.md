
genisoimage
rpm-build
epel-release
	/etc/rhsm/rhsm.conf
httpd
/opt/puppetlabs/puppet/bin/gem
	or install rubygems rpm 
enable proxy in iebfisa.sh 
subscription-manager register
subscription-manager attach --auto
update /etc/rhsm/rhsm.conf with proxy info 
subscription-manager attach --auto
yum install ruby-devel gcc make rpm-build rubygems
yum install --enablerepo=rhel-7-server-e4s-optional-rpms ruby-devel

gem install --no-ri --no-rdoc fpm --source http://rubygems.org



mkisofs -U -r -v -T -J -joliet-long -V "RHEL-7.5 Server.x86_64" -volset "RHEL-7.5 Server.x86_64" -A "RHEL-7.5 Server.x86_64" -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e images/efiboot.img -no-emul-boot -o ../iebfisa-rhel-75.iso .


gem install --no-ri --no-rdoc fpm --source http://rubygems.org


fpm -s dir -t rpm -C DL380Gen9 --name DL380Gen9 -v 1.0 --iteration 3.1.el7 --description "FW upd BIOS(01/22/2018), iLO(2.55) and SSA(6.30-0)" --url http://www.nex.com --license "none" --after-install=DL380Gen9/tmp/maintenance/after_install.sh .

--------------
infra 01p

httpd
gnuplot
collectl-utils/colplot install from github

change ^dir= to actual plots folder.
