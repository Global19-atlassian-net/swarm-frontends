# Kubernetes over Swarm

This folder contains a sample `compose.yml` file describing a Kubernetes deployment. It is not meant to be used in a production cluster but shows how easy it is to create and scale a Kubernetes cluster on top of Swarm.

The rationale behind this is that Swarm is lightweight enough to deploy additional orchestration tools on top. You begin with the regular docker experience and you can enhance this by adding orchestration/schedulers on top (`kubernetes`/`nomad`), etc.

Using Kubernetes on top of Swarm has many advantages: You can now use the *Replication Controller* alongside the *Pod* concept. Try to `rm` a container deployed with Kubernetes using Swarm, it will be still running thanks to Kubernetes.

Nice addition is that you can use the regular docker commands against the Kubernetes cluster: `inspect`/`logs`/`attach`, etc.

## Prerequisites

- Master version of Docker Compose: (see [Issue #2334](https://github.com/docker/compose/pull/2334))

- Latest version of Swarm: `1.0.0`

- Point Compose to the Swarm cluster if not local by setting `DOCKER_HOST`, `DOCKER_TLS_VERIFY`, `DOCKER_CERT_PATH` appropriately

- An *etcd, consul* or *zookeeper* cluster running for the overlay networking feature (this is not mandatory, the compose file can be tweaked to not use it but it's a nice addition to deploy across cloud providers).

See the [networking documentation](https://docs.docker.com/engine/userguide/networking/get-started-overlay/) to setup docker to use the Multi-Host networking features.

## Usage

To deploy your cluster, use:

`docker-compose --x-networking -f k8s-swarm.yml up`

You'll see logs from all the containers being deployed.

Modify your `.kube/config` to point to your kubernetes cluster. To do this, type `docker ps` and fetch the IP and port exposed from the `apiserver`.

You can then list nodes using:

`kubectl get nodes`

## Scale

To scale the kubelet instances you can now use:

`docker-compose --x-networking -f k8s-swarm.yml scale kubelet=2 proxy=2`

If you list nodes with `kubectl get nodes` you'll see the added node in the cluster

## Known limits / Possible enhancements

This is a base Compose file but it can be used to tweak a complete Production deployment, securing it with TLS, using distributed volumes (Ceph, GlusterFS) for the etcd cluster, etc. Feel free to propose enhancements! We'd like to see it working better.

Known issues are mainly the kubelet running in `privileged` mode. It is supposed to live on the Node so this might limit some capabilities/use cases. It does not prevent using the additional Kubernetes concepts on top of Swarm though.

A needed enhancement is also the use of ACLs to deploy the Kubernetes cluster with Swarm. This allows an admin user to deploy the cluster and regular users will not be able to delete the kubernetes components (It's in the work in [Swarm #1366](https://github.com/docker/swarm/pull/1366))