# AstarteC
AstarteC is a home project for vagrant multi machine creation supporting a configuration file for easier box setup, default values for fast run, /etc/hosts file automatic update, ssh key sharing and support for shell and ansible local provisioners.

## Requirements
Project is developed, executed and tested in a local workstation with Windows 10 OS. The below software needs to be in place:
 - Vagrant, version: 2.2.9
 - VirtualBox, version: 6.1.12

## Configuration
Before running, select the desired profile in config.yml, "use" key. In the example two profiles are available.
Default devB profile:
```sh
use: 'devB'
```
For the additional devA profile update config.yml:
```sh
use: 'devA'
```

## Execution
Check out the code, navigate to the project directory, optionally change the environment in config.yml and execute:
```sh
vagrant up
```
Setup process may take a while depending on the internet connection and workstation resources.

## Configuration file
The project details are controlled by config.yml file, there is no need for any change in vagrantfile for execution.
Selecting the active environment:
```sh
use: 'devB'
```
Default values that will be inherited to all boxes unless overlapped in environment configuration:
```sh
    vm:
      box: 'centos/7'
      ip: '192.168.50.10'
      memory: '512'
      cpus: '1'
      cpuexecutioncap: '50'
      provisioner:
        shell: false
        ansible_local: false
```
- **box**: Select the desired box name.
- **ip**: Select the base IP address. Unless overlapped in environment configuration, each box will be configured as part of a private network while incrementing its IP address by 1.
- **memory**: Select the default memory that will be allocated to each box.
- **cpus**: Select the default number of cpus (some boxes may require more than 1).
- **cpuexecutioncap**: Select the amount of time a host CPU spends to emulate a virtual CPU.
- **provisioner.shell**: Set by default to false. If enabled in environment configuration the provisioning/<environment>/shell/all.sh will be executed to all hosts that have this value as true, and provisioning/<environment>/shell/<hostname>.sh will be executed only to the specific host that has this value as true.
- **provisoner.ansible_local**: Set by default to false. If enabled in environment configuration the box will be assigned as ansible controller and will install and execute ansible using main.yml file and hosts inventory file located in provisioning/<environment>/ansible_local. Due to the vagrant box startup order it is generally recommended to configure the last vm as an ansible_local controller. If any vm, other than the last one, is configured as ansible_local controller a warning message will pop up during execution but the process will not be interrupted. If a vm is configured as ansible_local controller a set or private/public rsa keys will be randomly generated. The private key will be assigned to the controller vm and the public key will be configured to every node in order for ssh authentication using keys needed by ansible_local to work.

Environment specific configuration that overlaps default values:
```sh
  devA:
    - A1
    - A2
    - A3
  devB:
    B1:
      box: 'minimal/trusty64'
      memory: '256'
    B2:
      ip: '192.168.50.112'
      provisioner:
        shell: true
    B3:
    B4:
      cpus: '2'
      cpuexecutioncap: '75'
      provisioner:
        ansible_local: true
```

## Example environment devA
Update config.yml, use key:
```sh
use: 'devA'
```
Environment configuration for devA:
```sh
  devA:
    - A1
    - A2
    - A3
```
Execute vagrantfile
```sh
vagrant up
```
Since no overlapping configuration is provided the default values will be used and the below vms will be created:
```sh
hostname: A1
  box: centos/7
  ip: 192.168.50.11
  memory: 512
  cpus: 1
  cpuexecutioncap: 50
  provisioner:
    shell: false
    ansible_local: false
hostname: A2
  box: centos/7
  ip: 192.168.50.12
  memory: 512
  cpus: 1
  cpuexecutioncap: 50
  provisioner:
    shell: false
    ansible_local: false
hostname: A3
  box: centos/7
  ip: 192.168.50.13
  memory: 512
  cpus: 1
  cpuexecutioncap: 50
  provisioner:
    shell: false
    ansible_local: false
```
## Example environment devB
Update config.yml, use key:
```sh
use: 'devB'
```
Environment configuration for devB:
```sh
  devB:
    B1:
      box: 'minimal/trusty64'
      memory: '256'
    B2:
      ip: '192.168.50.112'
      provisioner:
        shell: true
    B3:
    B4:
      cpus: '2'
      cpuexecutioncap: '75'
      provisioner:
        ansible_local: true
```
Execute vagrantfile
```sh
vagrant up
```
The overlapping configuration will be taken into account, missing values will be set to default and the below vms will be created:
```sh
hostname: B1
  box: minimal/trusty64
  ip: 192.168.50.11
  memory: 256
  cpus: 1
  cpuexecutioncap: 50
  provisioner:
    shell: false
    ansible_local: false
hostname: B2
  box: centos/7
  ip: 192.168.50.112
  memory: 512
  cpus: 1
  cpuexecutioncap: 50
  provisioner:
    shell: true
    ansible_local: false
hostname: B3
  box: centos/7
  ip: 192.168.50.13
  memory: 512
  cpus: 1
  cpuexecutioncap: 50
  provisioner:
    shell: false
    ansible_local: false
hostname: B4
  box: centos/7
  ip: 192.168.50.14
  memory: 512
  cpus: 2
  cpuexecutioncap: 75
  provisioner:
    shell: false
    ansible_local: true
```
- **B1**: Uses a different box and limited memory resources. Provisioner ansible_local from B4 will create a ansible_local.txt  file under /home/vagrant.
- **B2**: Uses a custom ip 192.168.50.112 (instead of 192.168.50.12 that will be skipped). Also executes provisioning/devB/shell/all.sh and provisioning/devB/shell/B2.sh creating sample_all.txt and sample_single.txt under /home/vagrant. Provisioner ansible_local from B4 will create a ansible_local.txt  file under /home/vagrant.
- **B3**: Uses default values. Provisioner ansible_local from B4 will create a ansible_local.txt  file under /home/vagrant.
- **B4**: Uses 2 cpus, 75% cpu execution cap, has its public key shared to all other vms and installs and runs ansible mainl.yml and hosts from provisioning/devB/ansible_local creating an ansible_local.txt under /home/vagrant to B1, B2 and B3.
- **All vms**: File /etc/hosts is updated with the new ip and hostname entries

