# starlingx-workshop - Provisioning the Lab
 
The Ansible playbooks and roles in this directory will create as many Packet.com nodes are needed in which to run a particular workshop. Each student is, at this time, expected to get one physical node. That physical node will be created and configured to run StarlingX in a libvirt based virtual machine. Once this playbook completes students will have a place to start their workshop.

This is not intended for students to use. The instructors for the course would run this playbook to setup the lab for the students beforehand.

## Workshop Deployment

Clone this repository.

```
git clone https://github.com/ccollicutt/starlingx-workshop
```

cd into the Ansible directory.

```
cd starlingx-workshop/ansible
```

Create a virtual env and installed required Python pip packages.

```
virtualenv ~/venv/starlingx-workshop
. ~/venv/starlingx-workshop/bin/activate
pip install -r requirements.txt
```

Install Docker role.

```
ansible-galaxy install geerlingguy.docker
``` 

Ensure you have a packet cloud API token, project ID, and ssh key name and that they are exported to your session.

```
export PACKET_PROJECT_ID=<project id>
export PACKET_API_TOKEN=<token>
export PACKET_KEY_NAME=<key name>
```

At this point the inventory script should work.

```
./inventory/packet_net.py
```

Set the number of nodes to deploy (one node per student). Packet.com has regions. Sometimes it's not possible to get all the nodes needed for the workshop in one region. Thus, when specifying the number of nodes, regions can also be expressed in the yaml file. Configure the `packet_environments` variable in the `group_vars/all.yml` file.

Eg. The below would create 10 nodes in both `ewr1` and `nrt1` for a total of 20 nodes.

```
packet_environments:
  - region: "ewr1"
    num_nodes: "10"
  - region: "nrt1"
    num_nodes: "10"
```

Run the playbook.

*NOTE: It may take several minutes for the packet.com nodes to become available. A full run through a large number of nodes could take an hour or more.*

```
ansible-playbook all.yml
```

At this point you should have as many nodes as you requested ready to run `virsh start simplex-controller-0` on.

## Access Information

Once the `all.yml` playbook has completed without error, a local `access.txt` file will exist that lists out all of the IP addresses associated with the lab, the student user, and password for that node. This could be copied and pasted into an etherpad for the lab.

```
$ # Eg. output, only 2 nodes
$ cat access.txt 
(starlingx-workshop) [curtis@ash ansible]$ cat access.txt 
# Paste the below into https://etherpad.openstack.org/p/stx-workshop-access
1. Public IP: <public IP node 1>, User Name: student, Password: P@ssw0rd, OpenStack Horizon: http://<public IP node 1>:8080
2. Public IP: <public IP node 2>, User Name: student, Password: P@ssw0rd, OpenStack Horizon: http://<public IP node 2>:8080
```

Users can then ssh into the nodes with:

```
$ ssh student@<public IP 1>
$ # enter password of "P@ssw0rd"
```

## Options

* `hypervisor_use_apt_mirrors` - default is True, and will set to use apt mirrors in `sources.list`

## More than Two Workshops in the Same Packet.com Account

If for some reason we need to have more than two workshops running in the same Packet.com account, change the `workshop_tag` variable in the `packet-cloud` role so that each workshop uses a different tag. Then change the use of the `tag_stx_workshop` in the various top level playbooks. (This could be greatly improved.)

## Delete Infrastructure

Set `packet_node_state` to absent.

*NOTE: This will delete all of the workshop nodes in your Packet.com project!*

```
ansible-playbook provision-nodes.yml -e "packet_node_state=absent"
```