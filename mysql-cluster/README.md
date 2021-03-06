# redis-ansible-cluster
The demo sets up 6 redis instances, 1 corvus instance, and 1 instance to run tests.

## Installation (MacOS)

1. VirtualBox: `brew cask install virtualbox`
2. Vagrant: `brew cask install vagrant`
3. Vagrant Plugins: `vagrant plugin install vagrant-auto_network && vagrant plugin install vai`
4. Ansible: `brew install ansible`

## Create VMs

```
vagrant up
```

## Provision Redis Cluster Nodes

```
cd ansible
ansible-playbook provision_redis_cluster_nodes.yml
```

## Create Redis Cluster

```
ansible-playbook create_redis_cluster_config.yml
vagrant ssh redis-1
bash ./create_cluster
```

## Provision Corvus

```
ansible-playbook provision_corvus.yml
```

## Test corvus/redis cluster

```
ansible -m setup corvus1 -a 'filter=ansible_eth1'
```

Locate the ipv4 address of corvus

```
"ipv4": {
    "address": "10.55.0.8",
    "broadcast": "10.55.0.255",
    "netmask": "255.255.255.0",
    "network": "10.55.0.0"
},
```

```
redis-benchmark -h 10.55.0.8 -p 12345 -t set,get -n 100000 -c 20
```

There is also a very simple test script available for PHP that tests phpredis against corvus:

```
ansible-playbook provision_redis_test.yml
vagrant ssh redis-client
php redis-test.php
```

## Redis Cluster Reset
You can use ansible ad hoc commands to reset the cluster

```
cd ansible
ansible redis -m shell -a 'redis-cli FLUSHALL'
ansible redis -m shell -a 'redis-cli CLUSTER RESET'
```

## Add More Nodes To Redis Cluster
Best resource to add nodes in redis-cluster is found [here](https://redis.io/topics/cluster-tutorial)


## Debugging
Corvus logs are present at /var/log/syslog or by using `journalctl`

```
sudo journalctl -u corvus.service
```

You may run into issues after VMs have been shut down. These are usually fixed by re-running `vagrant provision` so that ansible has updated IP addresses and forcing fact gathering:

```
cd ansible
ansible-playbook gather_facts.yml
```