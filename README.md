# Elk-Project-2020
UCI Project - Elk Stack Deployment.


# Automated ELK Stack Deployment
The files in this repository were used to configure the network depicted below.

1[!alert]

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the Ansible folder may be used to install only certain pieces of it, such as Filebeat.

This document contains the following details:

* Description of the Topologu 
* Access Policies
* ELK Configuration
  - Beats in Use
  - Machines Being Monitored
* How to Use the Ansible Build

# Description of the Topology
The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.
Load balancing ensures that the application will be highly available, in addition to restricting traffic to the network.

Load balancing is a methodical and efficient distributor of network or application traffic across multiple servers in a shared network. The load balancer sits between client devices and backend servers, receiving and then distributing incoming requests to any available server capable of fulfilling particular request. 

An advantage of using a jump box, is the fact that it prevents all your networs VM's exposer to the public. We can do monitoring and logging on a single box. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the application and system infustructure.

Filebeat enables analysts to monitor files for suspicious changes.

Metricbeat makes it easy to collect specific information about the machines in the network.

The configuration details of each machine may be found below.



| Name | Function | IP Address | Operating System |
| --- | --- | --- | --- |
| Jump Box | Gateweay | 10.10.0.7 | Linux |
|Red Team VM 1 | Webserver  | 10.10.0.9 | Linux |
|Red Team VM 2 | Webserver  | 10.10.0.10 | Linux |
|Elk Stack VM | Monitoring  | 10.1.0.4 | Linux |


In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into the following availability zones:

  -Availability Zone 1: DVWA 1 + DVWA 2
  -Availability Zone 2: ELK

# Elk Server Configuration

The ELK VM exposes an Elastic Stack instance. Docker is used to download and manage an ELK container.

Rather than configure ELK manually, we opted to develop a reusable Ansible Playbook to accomplish the task. This playbook is duplicated below.

To use this playbook, one must log into the Jump Box, then issue: ansible-playbook install_elk.yml elk. This runs the install_elk.yml playbook on the elk host.
 
# Access Policies
The machines on the internal network are not exposed to the public Internet.

Only the jump box machine can accept connections from the Internet. Access to this machine is only allowed from the IP address 192.168.56.1

Machines within the network can only be accessed by each other. The DVWA 1 and DVWA 2 VMs send traffic to the ELK server.

A summary of the access policies in place can be found in the table below.



| Name | Publicly Accessible | Allowed IP Addresses |
| --- | --- | --- |
| Jump Box | Yes | 192.168.56.1 |
| Elk | No | 10.0.0.1-254 |
| DVWA 1 | No | 10.0.0.1-254 |
| DVWA 2 | No | 10.0.0.1-254 |



# Elk Configuration
Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...

TODO: What is the main advantage of automating configuration with Ansible?

The playbook implements the following tasks:

TODO: In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc.
...
...

The following screenshot displays the result of running docker ps after successfully configuring the ELK instance.

The playbook is duplicated below.

``` python
###
---
# install_elk.yml
- name: Configure Elk VM with Docker
  hosts: elkservers
  remote_user: elk
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
``` 

The following screenshot displays the result of running docker ps after successfully configuring the ELK instance.
Note: The following image link needs to be updated. Replace docker_ps_output.png with the name of your screenshot image file.


# Target Machines & Beats
This ELK server is configured to monitor the DVWA 1 and DVWA 2 VMs, at 10.10.0.9 and 10.10.0.10.

We have installed the following Beats on these machines:

* Filebeat
* Metricbeat
* Packetbeat

These Beats allow us to collect the following information from each machine:

Filebeat: Filebeat detects changes to the filesystem. Specifically, we use it to collect Apache logs.

Metricbeat: Metricbeat detects changes in system metrics, such as CPU usage. We use it to detect SSH login attempts, failed sudo escalations, and CPU/RAM statistics.

Packetbeat: Packetbeat collects packets that pass through the NIC, similar to Wireshark. We use it to generate a trace of all activity that takes place on the network, in case later forensic analysis should be warranted.

# Using the Playbook
In order to use the playbooks, you will need to have an Ansible control node already configured. We use the jump box for this purpose.

To use the playbooks, we must perform the following steps:

* Copy the playbooks to the Ansible Control Node
* Run each playbook on the appropriate targets

The easiest way to copy the playbooks is to use Git:

```python
$ cd /etc/ansible
$ mkdir files
#Clone Repository + IaC Files
$ git clone https://github.com/yourusername/project-1.git
#Move Playbooks and hosts file Into `/etc/ansible`
$ cp project-1/playbooks/* .
$ cp project-1/files/* ./files
```
This copies the playbook files to the correct place.

Next, you must create a hosts file to specify which VMs to run each playbook on. Run the commands below:

```python
$ cd /etc/ansible
$ cat > hosts <<EOF
[webservers]
10.10.0.9
10.10.0.10

[elk]
10.1.0.4
EOF
```
After this, the commands below run the playbook:

```python
$ cd /etc/ansible
$ ansible-playbook install_elk.yml elk
$ ansible-playbook install_filebeat.yml webservers
$ ansible-playbook install_metricbeat.yml webservers
```
To verify success, wait five minutes to give ELK time to start up.

Then, run: curl http://10.1.0.4:5601. This is the address of Kibana. If the installation succeeded, this command should print HTML to the console.

As a Bonus, provide the specific commands the user will need to run to download the playbook, update the files, etc.
