# rancher-swarm-weave

In this tutorial we’re going to show you how to provision a Docker Warm cluster with Rancher and how to 
visualize and monitor the cluster with Weave Scope.

### Rancher server setup

In order to install the Rancher server you'll need a Linux box with Docker 17.06 or later.
Deploy Rancher server with this command:

```bash
docker run -dp 8080:8080 --name=rancher --restart=unless-stopped rancher/server
```

Once the container is up you can access the Rancher UI from your browser on port 8080. The first thing you 
should do is to setup authentication. Navigate to the Access control section, chose the local authority for 
the Rancher Management interface and create an admin user with a password.

### Docker Swarm setup

In order to provision Docker Swarm with Rancher you have to create a new environment. 
In Rancher UI select _Manage Environments_ from the dropdown of environments, click on _Add Environment_ 
and select an environment template that has Swarm as the orchestration.

![env](https://github.com/stefanprodan/rancher-swarm-weave/blob/master/screens/add_env.png)

After the Swarm environment has been created, you can navigate to the environment 
by selecting the name of the environment in the environment’s dropdown. For Swarm to work correctly you'll 
have to create 3 hosts. From the _Infrastructure_ dropdown select _Hosts_ and click on _Add Host_.

When creating the hosts make sure they are in the same network and the host OS kernel has the IP_VS_NFCT 
module enabled. Using Ubuntu 16.04 LTS will the mainline kernel should work fine. You can check if the host 
OS is compatible with Docker Swarm by running the Moby [check-config.sh](https://github.com/moby/moby/blob/master/contrib/check-config.sh) script.

Once the hosts have been added, Rancher will automatically start the deployment of the 
infrastructure services including the Swarm components and the Portainer containers. 
You can see the progress of the deployment by accessing _Infrastructure Hosts_ from the menu.

![hosts](https://github.com/stefanprodan/rancher-swarm-weave/blob/master/screens/hosts.png)

When the health checks of all hosts are passing you can access _Portainer_ from the _Swarm_ dropdown. 
In Portainer UI you can see the Swarm cluster members by selecting _Swarm_ from the left side menu.

![portainer](https://github.com/stefanprodan/rancher-swarm-weave/blob/master/screens/portainer.png)

### Weave Scope setup

Now that you have a Docker Swarm cluster up and running you can start monitoring it with Weave Scope. 
You'll need a Weave Could service token, if you don't have a Weave token go 
to [Weave Cloud](https://cloud.weave.works/) and sign up for a Weave Cloud account. 

In order to visualize the Swarm cluster with Weave Cloud you have to deploy the Weave Scope container 
on each Swarm node. Like the Rancher infrastructure services, Scope needs to run with privileged mode and 
because Swarm services don't support privileged mode you'll have to use a one-shot service that will 
provision each node with a Scope container.

In Rancher UI navigate to _Swarm CLI_ and create the scope-launcher global service with the following command:  

```bash
docker service create --name scope-launcher --mode global --detach \
    --restart-condition none \
    --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
    weaveworks/scope-swarm-launcher \
    scope launch --service-token=<WEAVE-CLOUD-TOKEN>
```

In Rancher UI navigate to _Infrastructure containers_ and look for weavescope, 
you should see 3 containers running, one on each host:

![scope_list](https://github.com/stefanprodan/rancher-swarm-weave/blob/master/screens/scope_list.png)

The launcher will install Scope on each server and will exit. Using `--restart-condition none` we 
instruct Docker Swarm not to restart the service after it exits. 

![launcher_list](https://github.com/stefanprodan/rancher-swarm-weave/blob/master/screens/launcher_list.png)

With `--mode global` we make sure Scope will be automatically deployed on new servers 
as you add them to the swarm. 

Once the Scope containers are running on all hosts you can login into Weave Cloud and inspect your cluster.

![scope](https://github.com/stefanprodan/rancher-swarm-weave/blob/master/screens/scope.png)

With Weave Cloud Scope you can see your Rancher hosts, Docker containers and services in real-time. 
You can view metrics, tags and metadata of the running processes, containers or hosts. 
It’s the idea tool to visualize your Docker Swarm clusters and troubleshoot problems that may arise. 
Scope offers remote access to the Swarm’s nods and containers making it easy to diagnose issues 
in real-time.
