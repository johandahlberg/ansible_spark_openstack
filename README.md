Setting up a stand-alone spark cluster on OpenStack
===================================================

This describes how start a standalone [Spark](http://spark.apache.org/) cluster on open stack, using two [ansible](http://www.ansible.com) playbooks. This has been tested on the [Uppmax](http://www.uppmax.uu.se/) private cloud smog.

It will install spark and hdfs, and start the required services on the nodes. Please note that this is a proof-of-concept implementation, and that is is not ready for use in a production setting. Any pull requests to improve upon this to bring it closer to a production ready state are very much appreciated.

The open stack dymamic inventory code presented here is adapted from: https://github.com/lukaspustina/dynamic-inventory-for-ansible-with-openstack

How to start it?
-----------------
- Create a host from which to run ansible in your OpenStack dashboard and associate a floating IP to is so that you can `ssh` in to it.
- `ssh` to the machine you just created.
- Install the pre-requisites:
```
sudo apt-get install python-pip python-dev
sudo pip install ansible
sudo pip install python-novaclient
```
- Clone this repository:
```
git clone https://github.com/johandahlberg/ansible_spark_openstack.git
```
- Create a dir called `files` in the repo root dir and copy you ssh-keys (these cannot have a password) there. This is used to enable password-less ssh access between the nodes:
- Download you OpenStack RC file from the OpenStack dashboard (it's available under "Access & Security -> API Access") 
- Source your OpenStack RC file: `source <path to rc file>`, and fill in your OpenStack password. This will load information about you OpenStack Setup into your environment.
- Create the security group for spark. Since spark will start some services on random ports this will allow all tcp traffic within the security group:
```
nova secgroup-create spark "internal security group for spark"
nova secgroup-add-group-rule spark spark tcp 1 65535
```
- Setup the name of your network. `export OS_NETWORK_NAME="<name of your network>"` If you like you can add this to your OpenStack RC file, or set it in your `bash_rc`. (You can find the name of your network in your OpenStack dashboard)

- Edit the setup variables to fit your setup. Open `vars/main.yml` and setup the variables as explained there.
- One all the variables are in place you should now be able to create your instances:
```
ansible-playbook -i localhost_inventory --private-key=<your_ssh_key> create_spark_cloud_playbook.yml
```
- Then install spark on the nodes (I've noticed that sometimes it takes a while for the ssh-server on the nodes to start, so if you get an initial ssh-error, wait a few minutes and try again).
```
ansible-playbook -i openstack_inventory.py --private-key=<your_ssh_key> deploy_spark_playbook.yml
```
- Once this has finished successfully your spark cluster should be up and running! `ssh` into the spark-master node and try your new Spark cluster it by kicking of a shell. Now you're ready to enter into the Spark world. Have fun!
```
spark-shell --master spark://spark-master:7077 --executor-memory 6G
```

Tips
----
If you don't want to open the web-facing ports you can use ssh-forwarding to reach the web-interfaces, e.g

```
ssh -L 8080:spark-master:8080 -i <your key> ubuntu@<spark-master-ip>
```

Acknowledements
---------------
- Mikael Huss (@hussius) for sharing his insights on Spark and collaborating with me on this
- Zeeshan Ali Shah (@zeeshanali) for helping me get going with OpenStack

