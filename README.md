# rancher-swarm-weave

Rancher + Docker Swarm + Weave Cloud integration

# Run Weave Scope on Docker Swarm

```bash
docker service create --name scope-launcher --mode global --restart-policy=none \
    --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
    stefanprodan/scope-swarm-launcher \
    scope launch --service-token=<WEAVE-CLOUD-TOKEN>
```
