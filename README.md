Setting up Mesos using Vagrant
===

This contains two vagrant sandbox environment.

* standalone environment
* cluster environment (still buggy. please refer to [MESOS-435](https://issues.apache.org/jira/browse/MESOS-435?page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel))
  * mesos ships useful [mesos-ec2 scripts](https://github.com/apache/mesos/blob/master/docs/EC2-Scripts.textile). you can try it. 


Prerequisites
----
* VirtualBox: <https://www.virtualbox.org/>
* vagrant 1.2+: <http://www.vagrantup.com/>
* vagrant plugins
    * [vagrant-omnibus](https://github.com/schisamo/vagrant-omnibus)
          `$ vagrant plugin install vagrant-omnibus`
    * [vagrant-berkshelf](https://github.com/RiotGames/vagrant-berkshelf)
          `$ vagrant plugin install vagrant-berkshelf`
    * [vagrant-hosts](https://github.com/adrienthebo/vagrant-hosts)
          `$ vagrant plugin install vagrant-hosts`
    * [vagrant-cachier](https://github.com/fgrehm/vagrant-cachier)(optional)
          `$ vagrant plugin install vagrant-cachier`
 
Standalone Environment
----
### Setup a Mesos Virtual Box
It's so simple! It's time to get a cup of coffee because this may take some time.

    $ cd standalone
    $ vagrant up
    
### Mesos cluster in single node.
log in to the VM and you just need to hit below commands.

1. start a mesos master:

        $ vagrant ssh
        vagrant@mesos$ sudo mesos-master
1. start a mesos slave:

        $ vagrat ssh
        vagrant@mesos$ sudo mesos-slave --master=mesos:5050

If everything went well, you can see mesos web UI on: <http://localhost:5050>

### Mesos cluster managed with zookeeper
if you want to try mesos with zookeeper, which is responsible for managing master processes, you can try belows.

1. start zookeeper:

        $ vagrant ssh
        vagrant@mesos$ cd mesos/build/3rdparty/zookeeper-3.3.4/
        vagrant@mesos$ cp conf/zoo_sample.cfg conf/zoo.cfg
        vagrant@mesos$ sudo bin/zkServer.sh start

1. start mesos master with zookeeper:

        $ vagrant ssh
        vagrant@mesos$ sudo mesos-master --zk=zk://mesos:2181/mesos

1. start mesos slave with master managed with zookeeper:

        $ vagrant ssh
        vagrant@mesos$ sudo mesos-slave --master=zk://mesos:2181/mesos

### Try some example frameworks
please try below by following the [getting started](http://mesos.apache.org/gettingstarted/) document.

    $ vagrant ssh
    vagrant@mesos$ cd mesos/build
    vagrant@mesos$ src/test-framework --master=mesos:5050       # or --master=zk://mesos:2181/mesos
    vagrant@mesos$ src/example/java/test-framework mesos:5050   # or zk://mesos:2181/mesos
    vagrant@mesos$ src/example/python/test-framework mesos:5050 # or zk://mesos:2181/mesos

Multinode environment
----
### How to define cluster configurations
Cluster configuration is defined at `cluster.yml`.
You can edit the file to congigure cluster configurations.

```
# Cluster configurations
master_n: 1
slave_n : 1
zk_force: false

master_mem: 512
slave_mem : 1024
zk_mem    : 256

master_ipbase: "192.168.31."
slave_ipbase : "192.168.32."
zk_ipbase    : "192.168.33."
```

### Start & Stop Cluster
In multinode environment, `vagrant-mesos`, which is a helper script for controling mesos cluster is provided.

#### Launch a Cluster
This takes several minutes(may be 10 to 20 min.).  It's time for coffee.

```
$ cd multinodes
$ ./vagrant-mesos launch
```
#### Start/Stop a Cluster
```
$ cd multinodes
$ ./vagrant-mesos [start|stop]
```

#### Check a Status of a Cluster
```
$ cd multinodes
$ ./vagrant-mesos status
```

### Destroy a Cluster
this operations delete all VMs consisting mesos cluster.
```
$ cd multinodes
$ ./vagrant-mesos destroy
```

### Usage of `vagrant-mesos`
```
$ cd multinodes
$ ./vagrant-mesos -h
vagrant-mesos: vagrant wrapper helper script for controlling a mesos cluster.
Usage: vagrant-mesos [-h] command

   -h,  --help                       Print this help.

Available commands:
    destroy                          Destroy all VMs
    launch                           Equivalent to up and then start.
    start                            Start mesos cluster.
    stop                             Stop mesos cluser.
    provision                        Equivalent with 'vagrant provision'
    resume                           Equivalent with 'vagrant resume'
    status                           print status of VMs and cluster
    suspend                          Equivalent with 'vagrant suspend'
    up                               Equivalent with 'vagrant up'
```

