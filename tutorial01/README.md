# Tutorial 01 Installation of Chef Server on Ubuntu 14.04
------

### Prerequisite

1. OpenSSH to be installed into Ubuntu when you do a fresh install.
2. Patched the Ubuntu to the latest updates.

### Instruction to install Chef Server
#### Part 1. To disable apparmor,
Ensure that the response after running `sudo apparmor_status ` must be
`0 processes are in enforce mode` or `0 profiles are in enforce mode.` is returned,
AppArmor must be set to Complaining mode or disabled.

1. `sudo apparmor_status`
2. `sudo apt-get install apparmor-utils —y`
3. `sudo invoke-rc.d apparmor stop`
4. `sudo aa-complain /etc/apparmor.d/*` - To set AppArmor to Complaining mode or disabled.
5. `sudo update-rc.d -f apparmor remove`
6. `sudo apparmor_status`

#### Part 2. Installation of Chef Server
1. `wget https://packagecloud.io/chef/stable/packages/ubuntu/trusty/chef-server-core_12.4.1-1_amd64.deb/download`
2. `sudo dpkg -i download`
3. `sudo vim /etc/opscode/chef-server.rb`
```
server_name = "merino.com"
api_fqdn server_name
bookshelf['vip'] = server_name
nginx['url'] = "https://#{server_name}"
nginx['server_name'] = server_name
nginx['ssl_certificate'] = "/var/opt/opscode/nginx/ca/#{server_name}.crt"
nginx['ssl_certificate_key'] = "/var/opt/opscode/nginx/ca/#{server_name}.key"
```
4. `sudo chef-server-ctl reconfigure`
5. Run the following command.
```
sudo chef-server-ctl install chef-manage`
sudo chef-server-ctl reconfigure
sudo chef-manage-ctl reconfigure
```

6. if failed, run
```
sudo rm -rf /var/lib/apt/lists/*
sudo apt-get update
```

7. Run the following command.
```
sudo chef-server-ctl install opscode-reporting
sudo chef-server-ctl reconfigure
sudo opscode-reporting-ctl reconfigure
```

8. To create user, run the following command.
`sudo chef-server-ctl user-create ADMIN_USER_NAME ADMIN_FIRST_NAME ADMIN_LAST_NAME ADMIN_EMAIL ADMIN_PASSWORD --filename ADMIN_USER_NAME.pem`
e.g `sudo chef-server-ctl user-create sheep sheep sheep sheep@localhost.com P@ssw0rd --filename sheep.pem`
9.	`sudo chef-server-ctl org-create ORG_SHORT_NAME "ORG_LONG_NAME" --association_user ADMIN_USER_NAME`
e.g `sudo chef-server-ctl org-create sheep "sheep" --association_user sheep`
10. In windows, change the hostname by going to `C:\Windows\System32\drivers\etc`, edit th hosts file by adding this line
`192.168.198.131		sheep.com`
11. Go to https://192.168.198.131/login
12. download the starter kit. (Notes: made to reset the keys for organization and users. For step 8 and 9, you can create it in the UI.)

### Reference

1. https://docs.chef.io/install_server_pre.html#apparmor
2. https://learn.chef.io/install-and-manage-your-own-chef-server/linux/install-chef-server/install-chef-server-using-your-hardware/
