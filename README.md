# contrail-container-deployer
## instructions

### get playbook    

```
git clone http://github.com/michaelhenkel/contrail-container-deployer
```

### configuration    

The playbook consists of three main plays:    
- install virtual machines    
- configure/install required software on virtual machines    
- create containers    
All three plays can be enabled/disabled:    

```
vi inventory/group_vars/all.yml
---
BUILD_VMS: true #will build virtual machines, true/false
CONFIGURE_VMS: true #will configure virtual machines, true/false
CREATE_CONTAINERS: true #will create containers, true/false
```

#### VM configuration

The pre-requisites for building the virtual machines are:    
- kvm    
- a virsh network to which the VMs can be connected    

The containers VMs must be assigned to a KVM host:    
```
vi inventory/hosts
hypervisors:
  hosts:
    10.87.64.31:        #KVM host1
      bridge: br1       #virsh network to which the container VMs will be connected to
      container_vms:    #container VMs assigned to the KVM host
        192.168.1.100:
    10.87.64.32:
      bridge: br1
      container_vms:
        192.168.1.101:
    10.87.64.33:
      bridge: br1
      container_vms:
        192.168.1.102:
```

The container VMs configuration is done in:    
```
vi inventory/group_vars/all.yml
CENTOS_DOWNLOAD_URL: http://cloud.centos.org/centos/7/images/
CENTOS_IMAGE_NAME: CentOS-7-x86_64-GenericCloud-1710.qcow2.xz
CONTAINER_VM_ROOT_PWD: contrail123
CONTAINER_VM_CONFIG:
  root_pwd: contrail123
  vcpu: 4
  vram: 16384
  vdisk: 100G
  network:
    subnet_prefix: 192.168.1.0
    subnet_netmask: 255.255.255.0
    gatway: 192.168.1.1
    nameserver: 10.84.5.100
    ntpserver: 192.168.1.1
    domainsuffix: local
```

#### Contrail configuration

The minimal configuration requires the cluster node  ip addresses,    
the contrail version and docker registry:    
```
inventory/group_vars/container_vms.yml
CONTAINER_REGISTRY: michaelhenkel
contrail_configuration:
  CONTRAIL_VERSION: 4.1.0.0-4
  CONTROLLER_NODES: 192.168.1.100,192.168.1.101,192.168.1.102
  CLOUD_ORCHESTRATOR: kubernetes
```

The role assignment is done here:    
```
container_vms:
  hosts:
    192.168.1.100:
      ansible_ssh_pass: contrail123
      roles:
        configdb:
        config:
        control:
        webui:
        analytics:
        analyticsdb:
        k8s_master:
        vrouter:
    192.168.1.101:
      ansible_ssh_pass: contrail123
      roles:
        configdb:
        config:
        control:
        webui:
        analytics:
        analyticsdb:
        k8s_master:
        vrouter:
    192.168.1.102:
      ansible_ssh_pass: contrail123
      roles:
        configdb:
        config:
        control:
        webui:
        analytics:
        analyticsdb:
        k8s_master:
        vrouter:
```
## Execution

```
ansible-playbook -i inventory/ playbooks/deploy.yml
```
