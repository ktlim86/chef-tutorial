# Tutorial 03 Bootstrap Your Node
------

### Instructions
On Window workstation, use powershell to
1. Navigate to your `PATH_TO_CHEF_KITS/chef-repo` in command prompt
2. Make a directory by typing this
`mkdir cookbooks`
3. Run this command
`chef generate cookbook cookbooks/hello_chef_server`
4. Go to the default.rb and edit
```
file "#{Chef::Config[:file_cache_path]}/hello.txt" do
  content 'Hello, Chef server!'
end
```
5. `knife cookbook upload hello_chef_server`
6. `knife cookbook list`
7. Bootstrap your node by running this command
`knife bootstrap windows winrm 192.168.198.132 --winrm-user Administrator --winrm-password 'P@ssw0rd' --node-name node1 --run-list 'recipe[hello_chef_server]' --winrm-transport ssl --winrm-ssl-verify-mode verify_none`
8. verify it by going to the Windows Node and under `C:\chef\cache`, you will see a `hello.txt`.
9. Open it and you will see the
`Hello, Chef server!``

### Additional Notes
1. you must set the host in the `hosts` file to point to the chef server
in Windows, edit the hosts file in `C:/Windows/System32/drivers/etc`
`CHEF_SERVER_IP		CHEF_SERVER_HOSTNAME`
e.g
`192.168.198.132		fenix.com`
2. the workstation must be in the same network as the node1
3. You need to setup a DHCP server to dynamically allocate IP address to the server first. Otherwise, the provisioned VM will be in the automatically private IP address and the chef will not be able to reach the server and
configure the VM via ssh/winrm.
4. You must disable the IPv6. Otherwise, Chef has problem contacting the node as it utilized IPv6. To configure,
	For Windows node,
		1. Go to `regedit`
		2. Locate `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters\`
		3. Look for `DisabledComponent`s. If there isn't, create one `DWORD` and named it `DisabledComponents`. The value will be `ffffffff`.
		4. Restart it.
		5. Go to command prompt and type `'ipconfig'`. Make sure there is no IPv6 showing.
		Reference: https://support.microsoft.com/en-sg/kb/929852
5. If you want to install a chef using a particular version, store the chef installer in `C:/software`. When you provision,
find `install_msi_url` in `convergence_options` and put it as `file:///C:/software/chef-client-12.9.38-1-x64.msi` .
6. You must run the script to reset the winrm everytime you provision a new vm.
