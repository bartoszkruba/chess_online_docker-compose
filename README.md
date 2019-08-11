# Chess Online - docker-compose files
Docker compose .yml file for Chess online project

## How To Run:

### Backend Development Configuration:
Clone docker-compose repository from GitHub  
```shell script
$ git pull https://github.com/bartoszkruba/chess_online_docker-compose.git    
```

Then cd into **dev** folder


Run docker-compose file though following command:

```shell script
$ docker-compose up
```

By default server binds to port **8080** but you can change this by editing docker-compose file and adding following property: 
```
SERVER_PORT: {your_port} 
``` 

### Backend Production Configuration:

#### Running through Docker Swarm Stacks

Make sure you that you have enabled Docker Swarm mode and everything is configured properly (You can find more information about how to do it [here](https://docs.docker.com/engine/swarm/swarm-mode/))

#### etcb configuration

The DB stack is deployed using Galera Cluster docker image. This allows scaling up the MYSQL cluster out on more more than one node (or scaling back by removing nodes). The storage will be persisted across multiple nodes and if one of them goes down no data will be lost. 

Galera image requires  running etcd cluster installed on each of the Docker physical host.

How to configure etcd (CentOS 7):

1: Install etcd package:
```shell script
yum install etcd
```

2: Edit configuration
```shell script
$ vim /etc/etcd/etcd.conf
```

Should look like this:
```
ETCD_NAME=etcd1
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://{node_ip}:2380"
ETCD_INITIAL_CLUSTER="etcd1=http://192.168.55.111:2380,etcd2=http://192.168.55.112:2380,etcd3=http://192.168.55.113:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-1"
ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379"
```
3: Start the service:
```shell script
$ systemctl enable etcd
$ systemctl start etcd
```

4: Verify cluster status
```shell script
$ etcdctl cluster-health
```

You can find more info about edcb and Galera Cluste configuration [here](https://severalnines.com/blog/mysql-docker-deploy-homogeneous-galera-cluster-etcd)

#### Deploying on Docker Swarm Stacks

Clone docker-compose repository from GitHub  
```shell script
$ git pull https://github.com/bartoszkruba/chess_online_docker-compose.git    
```

Then cd into **prod** folder.  

Edit **docker-compose** file and change credentials for admin account, MySQL database and RabbitMQ. 
 
Default configuration is 2 backend services but you can change this setting to as many application replicas as you want. 

Add your nodes ip addresses to "DISCOVERY_SERVICE" environment variable inside mysql-galera configuration.  
  
Example:  
```
DISCOVERY_SERVICE: '192.168.55.111:2379,192.168.55.112:2379,192.168.55.207:2379'
```
Recommended setup for high availability is at least 3 Gale Cluster replicas
  
Deploy Docker stock by following command:
```shell script
$ docker stock deploy -c docker-compose.yml {your_stack_name}
```
