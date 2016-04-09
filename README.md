SC16: Experiments and Results
=============================

This repository has our experiments for our SuperComputing '16 submission. 

Install
-------

These experiments were run on [CloudLab](https://www.cloudlab.us). We have a 6-node Centos7.1 baremetal setup, so you can instantiate a cluster from our [GassyFS Profile](https://www.cloudlab.us/p/5fd60b18-f5d0-11e5-b570-99cadac50270). Select the Clemson cluster, since these nodes have infiniband. On all nodes:

1. Setup passwordless SSH and sudo

2. Install [Docker](https://docs.docker.com/engine/installation/)

3. Add EPEL

   ```bash
   wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm 
   sudo rpm -ivh epel-release-7-5.noarch.rpm
   sudo yum update -y
   ```

4. Setup up unlimited open files:

   ```bash
   echo "* soft memlock unlimited" | sudo tee -a /etc/security/limits.conf
   echo "* hard memlock unlimited" | sudo tee -a /etc/security/limits.conf
   ulimit -l unlimited
   ```

Quickstart
----------

1. Start an experiment master (a container with Ansible v2.0.1.0):

   ```bash
  ./emaster.sh
   ```

2. Choose an experiment and setup the hosts:

   ```bash
   [EXPERIMENT_MASTER] cd figure1/experiment-bamsort/
   [EXPERIMENT_MASTER] vim inventory
   [EXPERIMENT_MASTER] ./run.sh
   ```

Troubleshooting
---------------

Check to make sure everything installed smoothly:

   ```bash
   # Should return unlimited
   $ ulimit -l

   # Should show no running images
   $ docker ps 
   ```

If you still have problems, checkout the `bootstrap-infiniband.sh` file in the [srl-roles](https://github.com/systemslab/srl-roles) repository.

EOF 