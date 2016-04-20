# Tutorial 02 Preparation and Bootstrap of Windows Node
------

### Part 1: Preparation of window node

On Window Node,
1. Run the following command to create a self-signed certificate. Replace NODE_HOSTNAME with the IP address or FQDN that you can reach from your workstation.
`$cert = New-SelfSignedCertificate -CertStoreLocation Cert:\LocalMachine\My -DnsName 'NODE_HOSTNAME'`
`e.g $cert = New-SelfSignedCertificate -CertStoreLocation Cert:\LocalMachine\My -DnsName '192.168.198.132'`
2. Delete any existing listener by running `winrm delete winrm/config/Listener?Address=*+Transport=HTTPS`. If you receive an error, that's likely OK. An error will typically indicate that there was no existing HTTPS listener.
3. `New-Item -Address * -Force -Path wsman:\localhost\listener -Port 5986 -HostName ($cert.subject -split '=')[1] -Transport https -CertificateThumbPrint $cert.Thumbprint`
4. Run these commands to allocate 1GB of memory to each WinRM shell and set the timeout period to 30 minutes.
```
Set-Item WSMan:\localhost\Shell\MaxMemoryPerShellMB 1024
Set-Item WSMan:\localhost\MaxTimeoutms 1800000
```
5. Run the following the command to enable the firewall to allow anyone to use the WinRM. `netsh advfirewall firewall add rule name="WinRM-HTTPS" dir=in localport=5986 protocol=TCP action=allow`
6. Disable IPv6 by running the command.
For Windows node,
  1. Go to `regedit`
  2. Locate `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters\`
  3. Look for `DisabledComponent`s. If there isn't, create one `DWORD` and named it `DisabledComponents`. The value will be `ffffffff`.
  4. Restart it.
  5. Go to command prompt and type `'ipconfig'`. Make sure there is no IPv6 showing.

#### Additional Notes:
1. After you completed this step, you can turn this into a VM template.
2. If you wish to fix a particular version of chef-client, create a directory called `C:\softwares` and place the `chef-client-12.9.38-1-x64.msi`. Then under the `install_msi_url` in `convergence_options` and put it as `file:///C:/software/chef-client-12.9.38-1-x64.msi`.
3. The reason why IPv6 needs to be disabled is to ensure that Chef can contact the provisioned VM via IPv4. If this is not done, you will notice on the console/terminal that the Chef is using IPv6 to contact the provisioned VM.
4. Take note that everytime a VM is provisioned, reset the WinRM by running Step (1) - (4). The script will be written as a powershell script.  The script below add in a code to get the IPv4 address. Feel free to modify for your own needs.

```
Write-Host "Configuring WIN-HTTPS"
$ipconf = ipconfig
$ipconf = $ipconf -match "IPv4"
$ipconf = $ipconf.split(":")[1].Substring(1)
$cert = New-SelfSignedCertificate -CertStoreLocation Cert:\\LocalMachine\\My -DnsName $ipconf

winrm delete winrm/config/Listener?Address=*+Transport=HTTPS
New-Item -Address * -Force -Path wsman:\\localhost\\listener -Port 5986 -HostName ($cert.subject -split '=')[1] -Transport https -CertificateThumbPrint $cert.Thumbprint
Set-Item WSMan:\\localhost\\Shell\\MaxMemoryPerShellMB 1024
Set-Item WSMan:\\localhost\\MaxTimeoutms 1800000
```

### Part 2. Using prepare node to bootstrap

On your workstation,
1. `knife wsman test HOSTNAME --manual-list --winrm-transport ssl`
2. On your `PATH_TO_CHEF_KITS/.chef`, open `knife.rb`:
add in the following line:
`ssl_verify_mode			 :verify_none`
3. Run the following command
`knife wsman test 192.168.198.132 --manual-list --winrm-transport ssl --winrm-ssl-verify-mode verify_none`
```
Connected successfully to 192.168.198.132 at https://merino.com:5986/wsman
```
Make sure the workstation is on the same network as the node. Otherwise it will return
```
WARNING: Failed to connect to 192.168.198.132 at https://192.168.4.50:5986/wsman.
ERROR: Failed to connect to 1 nodes.
```

4. Go to https://CHEF_SERVER_IP and login.
5. You should be able to see the windows node in the Node tab.


### Reference
1. https://support.microsoft.com/en-sg/kb/929852
2. https://learn.chef.io/install-and-manage-your-own-chef-server/linux/manage-a-node-on-your-chef-server/
3. https://learn.chef.io/manage-a-node/windows/get-a-node-to-bootstrap/
4. https://learn.chef.io/manage-a-node/windows/bootstrap-your-node
