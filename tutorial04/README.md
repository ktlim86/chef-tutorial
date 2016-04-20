# Tutorial 04 Provisioning from vSphere vCenter
------

To allow provisioning of VM, you must carry out the following procedures before you can do anything.

### Pre-requisite:
1. [chef-provisioning](https://github.com/chef/chef-provisioning)
2. [chef-provisioning-vsphere](https://github.com/CenturyLinkCloud/chef-provisioning-vsphere)
3. DHCP server to assign address
4. The workstation must be on the same network as the provisioned VM
5. The VM template must have WinRM enabled before turning it into a VM template. Refer to [Tutorial 02](tutorial02/).
6. Stable network

### Instructions
Supposed you want to provision a VM, the configuration will be as shown below in the `provision.rb`.
```
#
# Cookbook Name:: setup_vm
# Recipe:: default
#
# Copyright 2016, YOUR_COMPANY_NAME
#
# All rights reserved - Do Not Redistribute
#
# Reference: https://github.com/CenturyLinkCloud/chef-provisioning-vsphere#kitchen-driver
require 'chef/provisioning'
require 'chef/provisioning/vsphere_driver'

chef_gem 'chef-provisioning-vsphere' do
  action :install
  compile_time true
end

with_vsphere_driver host: 'VCENTER_IP_ADDRESS',
  insecure: true,
  user:     'VCENTER_USER',
  password: 'VCENTER_PASSWORD'

with_machine_options :bootstrap_options => {
  use_linked_clone: true,
  num_cpus: 4,
  host: 'ESXI_HOST_IP',
  memory_mb: 4096,
  network_name: ["NETWORK_LABEL"],
  datastore: "DATASTORE_NAME",
  datacenter: 'VCENTER_DATACENTRE_NAME',
  template_name: 'VM_TEMPLATE',
  customization_spec: {
    ipsettings: {
      ip: 'PROVISION_VM_IP',
      subnetMask: 'SUBNET_MASK',
      gateway: ["GATEWAY"],
      dnsServerList: ['DNS1','DNS2']
    },
    win_time_zone: 215,
    domain: 'local',
    org_name: 'my_org',
    product_id: '',
    hostname: 'HOSTNAME'
  },
  :ssh => {
    :user => 'PROVISIONED_VM_USER',
    :password => 'PROVISIONED_VM_PASSWORD',
    :paranoid => false,
  }
},
# You may choose to specify the convergence_options.
convergence_options: {
  ssl_verify_mode: :verify_none, # Specify this to bypass the authentication with Chef Server if this is a on-premise deployment and not cloud chef server.
  install_msi_url: "file:///C:/software/chef-client-12.8.1-1-x64.msi" # specify this if you want to fix a version of chef-client.
}

machine "HOSTNAME" do
  run_list ['setup_vm::reset_winrm','setup_vm::install_python'] # you can specify any recipe that you want to run.
  #action :setup # by default, action is converge. setup means you choose to install only chef-client on the machine without installing any software.
end

```

1. Type this command at the chef-repo directory.
For example, if the directory is at `C:\Users\chef\chef-starter-kits\chef-repo`
`chef-client -o 'setup_vm::provision' -c .\.chef\knife.rb`.

### Additional Notes
1. Install a DHCP server to assign IP address to the provisioned VM. If you don't, you will encounter an error where the chef is not able to provisioned the VM due to uncontactable IP address. This is because the provisioned VM cannot contact DHCP to assign an IP address and therefore an automatic private IP address will be assigned.
2. Your workstation that issued the provisioned VM needs to be in the same network as the provisioned VM. The workstation cannot have two simulataneous network card otherwise chef cannot contact the provisioned VM.
3. If you wish to fix a particular Chef client version, then under the `install_msi_url` in `convergence_options` and put it as `file:///C:/software/chef-client-12.9.38-1-x64.msi` in the VM template.
4. You need to carry out perform basic system preparation on the VM that you want to use as a VM template. Refer to [Tutorial 02](tutorial02/) on Part 1: Preparation of window node.

### Reference:
1.  https://learn.chef.io/install-and-manage-your-own-chef-server/linux/manage-a-node-on-your-chef-server/
2. https://learn.chef.io/manage-a-node/windows/get-a-node-to-bootstrap/
3. https://learn.chef.io/manage-a-node/windows/bootstrap-your-node/
4. https://learn.chef.io/manage-a-web-app/windows/create-the-cookbook/
5. https://learn.chef.io/manage-a-node/windows/set-up-your-chef-server/
6. https://learn.chef.io/install-and-manage-your-own-chef-server/linux/
7. https://learn.chef.io/install-and-manage-your-own-chef-server/linux/install-chef-server/install-chef-server-using-your-hardware/
8. https://docs.chef.io/chef_overview.html
9. https://docs.chef.io/install_server_pre.html#apparmor
10. https://learn.chef.io/local-development/windows/get-set-up/get-set-up-vagrant/
