# Accumulo on Amazon Linux via Ansible

This directory contains a set of Ansible playbooks and configuration
file templates for setting up and running an Accumulo cluster on EC2
Amazon Linux (default AMI) instances.  As Amazon Linux is
Red-Hat--based, and we do not use EC2-specific Ansible features, these
playbooks may be easily portable to CentOS.

## Prerequisites

To run these scripts you must have a modern Ansible---at least
version 2, maybe 2.2 is required---installed on your local machine.
It may be easiest to install Ansible locally via pip:

```bash
pip install --user --upgrade ansible
```

We require one or more EC2 instances running Amazon Linux (the
default AMI) to be launched.  Each instance should have a large root
volume.  The instances should have open network communication between
themselves, and should all be accessible via the same SSH key.

As a convention, we create an
[Ansible inventory](http://docs.ansible.com/ansible/intro_inventory.html)
describing the EC2 instances' roles in the cluster in the current
directory, in the `hosts` file.  The roles each EC2 instance plays are
represented by its membership in specific Ansible groups (one or more
per instance).  Some groups should only have one instance, while some
may have several.  Instances are listed by their unqualified private
hostnames.  Note,
[as described in the documentation](https://zookeeper.apache.org/doc/r3.4.9/zookeeperStarted.html#sc_RunningReplicatedZooKeeper),
Zookeeper is most stable with an odd number of servers, so an odd
number of hosts should be listed in the `zookeepers` group.
See the example `hosts` file
[hosts.one-node.example](/accumulo/hosts.one-node.example) for a
single-node deployment and
[hosts.four-node.example](/accumulo/hosts.four-node.example) for a
four-node deployment.
To create the `hosts` file for your setup, we recommend copying one of
these and editing it.

## Installation

A configuration file `secrets.yml` must now be created in the current
directory.  This file is a simple
[YAML](http://yaml.org/spec/1.2/spec.html)
file containing the Accumulo root password
as well as the Accumulo instance secret.
See the example `secrets.yml` file
[secrets.yml.example](/accumulo/secrets.yml.example).

Next generate a passwordless SSH key for the `accumulo` user:

```bash
ssh-keygen -t rsa -f id_accumulo_rsa -N ''
```

Now run the install playbook, replacing `$HOME/.ssh/ec2.pem`
with SSH key used to access your EC2 instances:

```bash
ansible-playbook --private-key=$HOME/.ssh/grid_kp.pem -i hosts install.yml
```

Note, when connecting to instances for the first time, you may be
prompted to accept their key fingerprints (as in normal SSH
usage).

The install playbook should work as desired on a partially-installed
or fully-installed cluster (making only the necessary changes);
that is, it is idempotent.

Finally, to initialize the accumulo cluster, run the initialize
playbook, replacing `$HOME/.ssh/ec2.pem`
with SSH key used to access your EC2 instances.  Note, unlike the
install playbook, this playbook is not idempotent: it will likely
break a running cluster.

```bash
ansible-playbook --private-key=$HOME/.ssh/grid_kp.pem -i hosts initialize.yml
```

## Startup

After installation, to start Accumulo (and HDFS and Zookeeper),
run the start playbook, replacing `$HOME/.ssh/ec2.pem`
with SSH key used to access your EC2 instances:

```bash
ansible-playbook --private-key=$HOME/.ssh/grid_kp.pem -i hosts start.yml 
```

## Shutdown

To stop Accumulo (and HDFS and Zookeeper),
run the stop playbook, replacing `$HOME/.ssh/ec2.pem`
with SSH key used to access your EC2 instances:

```bash
ansible-playbook --private-key=$HOME/.ssh/grid_kp.pem -i hosts stop.yml 
```

## Reset

To stop all Accumulo-related processes (including HDFS and Zookeeper)
immediately and delete all data from HDFS and all logs,
run the reset playbook, replacing `$HOME/.ssh/ec2.pem`
with SSH key used to access your EC2 instances:

```bash
ansible-playbook --private-key=$HOME/.ssh/grid_kp.pem -i hosts reset.yml
```

## References

### Accumulo, Zookeeper, Hadoop, HDFS

* [Hadoop single-node cluster setup](https://hadoop.apache.org/docs/r2.7.3/hadoop-project-dist/hadoop-common/SingleCluster.html)
* [Hadoop multi-node cluster setup](https://hadoop.apache.org/docs/r2.7.3/hadoop-project-dist/hadoop-common/ClusterSetup.html)
* [Zookeeper getting started](https://zookeeper.apache.org/doc/r3.4.9/zookeeperStarted.html)
* [Accumulo user manual](https://accumulo.apache.org/1.8/accumulo_user_manual)
* [Accumulo installation](https://github.com/apache/accumulo/blob/1.8/INSTALL.md)
* Accumulo example configurations (in particular `conf/examples/3GB/native-standalone`, in the [Accumulo distribution](https://accumulo.apache.org/downloads/))

### Ansible

* [Jinja2 templates](http://jinja.pocoo.org/docs/dev/templates/)
* [Ansible getting started](http://docs.ansible.com/ansible/intro_getting_started.html)
* [Ansible intro to playbooks](http://docs.ansible.com/ansible/playbooks_intro.html)
  * As noted in this documentation,
    Ansible output is greatly improved when the cowsay package is
    installed.  This increased functionality is optional; cowsay can be
    installed on Red-Hat--based systems with: `sudo yum install cowsay`
