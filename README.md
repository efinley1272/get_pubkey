# get_pubkey
#if ruby isn't already installed
apt install -y ruby
#or
yum install -y ruby

cp get_pubkey /usr/local/sbin/
chown root:root /usr/local/sbin/get_pubkey
chmod 755 /usr/local/sbin/get_pubkey


#add or modify the following lines in /etc/ssh/sshd_config to match
PermitRootLogin prohibit-password
AuthorizedKeysFile      .ssh/authorized_keys.
AuthorizedKeysCommand /usr/local/sbin/get_pubkey %u keyserver1.veracitynetworks.com keyserver2.veracitynetworks.com keyserver3.veracitynetworks.net
AuthorizedKeysCommandUser nobody


systemctl restart sshd

#on centos
#try to log into the centos server with your key, it will fail because of selinux (if it's turned on)
#you can see why by

yum install setroubleshoot
sealert -a /var/log/audit/audit.log

#you'll see that the first failure is that the ruby script is not allowed to run the hostname command, so we fix that with:
ausearch -c 'hostname' --raw | audit2allow -M my-hostname
semodule -i my-hostname.pp

#now try to log in again.  it'll fail again, this time because it doesn't want to allow ruby to make an outgoing connection to the keyserver.  so we fix that with:
ausearch -c 'ruby' --raw | audit2allow -M my-ruby
semodule -i my-ruby.pp

#now try to log in again, it should work.
