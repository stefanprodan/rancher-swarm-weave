# rancher-swarm-weave

Rancher + Docker Swarm + Weave Cloud integration

### Rancher server setup

In order to install the Rancher server you'll need a Linux box with Docker 17.06 or later.
Deploy Rancher server with this command:

```bash
docker run -dp 8080:8080 --name=rancher --restart=unless-stopped rancher/server
```
Once the container is up you can access the Rancher UI from your browser on port 8080. The first thing you 
should do is to setup authentication. Navigate to the Access control section, chose the local authority for 
the Rancher Management interface and create an admin user with a password.

### Swarm cluster setup

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
You can see the progress of the deployment by accessing the by selecting _Infrastructure Hosts_ from the menu.

![hosts](https://github.com/stefanprodan/rancher-swarm-weave/blob/master/screens/hosts.png)

When the health checks of all hosts are passing you can access _Portainer_ from the _Swarm_ dropdown. 
In Portainer UI you can see the Swarm cluster members by selecting _Swarm_ from the left side menu.

![portainer](https://github.com/stefanprodan/rancher-swarm-weave/blob/master/screens/portainer.png)

### Run Weave Scope on Docker Swarm

```bash
docker service create --name scope-launcher --mode global --restart-condition none --detach true\
    --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
    stefanprodan/scope-swarm-launcher \
    scope launch --service-token=<WEAVE-CLOUD-TOKEN>
```
