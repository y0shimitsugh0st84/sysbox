# Sysbox User Guide: Functional Limitations

This document describes functional restrictions and limitations of Sysbox and
system containers.

## Contents

-   [Docker Restrictions](#docker-restrictions)
-   [Kubernetes Restrictions](#kubernetes-restrictions)
-   [System Container Limitations](#system-container-limitations)
-   [Sysbox Functional Limitations](#sysbox-functional-limitations)

## Docker Restrictions

This section describes restrictions when launching containers with Docker +
Sysbox.

### Support for Docker's `--privileged` Option

Sysbox system containers are incompatible with the Docker `--privileged` flag.

The raison d'être for Sysbox is to avoid the use of (very insecure) privileged
containers yet enable users to run any type of software inside the container.

Using the Docker `--privileged` + Sysbox will fail:

```console
$ docker run --runtime=sysbox-runc --privileged -it alpine
docker: Error response from daemon: OCI runtime create failed: container_linux.go:364: starting container process caused "process_linux.go:533: container init caused \"rootfs_linux.go:67: setting up ptmx caused \\\"remove dev/ptmx: device or resource busy\\\"\"": unknown.
ERRO[0000] error waiting for container: context canceled
```

### Support for Docker's `--userns=host` Option

When Docker is configured in userns-remap mode, Docker offers the ability
to disable that mode on a per container basis via the `--userns=host`
option in the `docker run` and `docker create` commands.

This option **does not work** with Sysbox (i.e., don't use
`docker run  --runtime=sysbox-runc --userns=host ...`).

Note that usage of this option is rare as it can lead to the problems as
described [in this Docker article](https://docs.docker.com/engine/security/userns-remap/#disable-namespace-remapping-for-a-container).

### Support for Docker's `--pid=host` and `--network=host` Options

System containers do not support sharing the pid or network namespaces
with the host (as this is not secure and it's incompatible with the
system container's user namespace).

For example, when using Docker to launch system containers, the
`docker run --pid=host` and `docker run --network=host` options
do not work with system containers.

### Support for Exposing Host Devices inside System Containers

Sysbox does not currently support exposing host devices inside system
containers (e.g., via the `docker run --device` option).

## Kubernetes Restrictions

This section describes restrictions when launching containers with Kubernetes +
Sysbox.

### Pods limited to 16 per-node on Sysbox-CE

Pods launched with the Sysbox Community Edition are **limited to \*\*16 pods per worker node\*\***.

Once this limit is reached, new pods scheduled on the node will remain in the
"ContainerCreating" state. Such pods need to be terminated and re-created once
there is sufficient capacity on the node.

#### ** --- Sysbox-EE Feature Highlight --- **

With Sysbox Enterprise (Sysbox-EE) this limitation is removed, as it's designed
for greater scalability. Thus, you can launch as many pods as will fit on the
Kubernetes node, allowing you to get the best utilization of the hardware.

Note that the number of pods that can be deployed on a node depends on many
factors such as the number of CPUs on the node, the memory size on the node, the
the amount of storage, the type of workloads running in the pods, resource
limits on the pod, etc.)

### Privileged pods are not allowed

The pod's security context must not have the `privileged: true` attribute.

The raison d'être for Sysbox is to avoid the use of (very insecure) privileged
containers yet enable users to run any type of software inside the container.

### Sharing Linux Namespaces with the Host is not allowed

The pod's spec must not share Linux namespaces with the host, as this breaks
container isolation. Thus avoid setting these in the pod's spec:

```yaml
hostNetwork: true
hostIPC: true
hostPID: true
```

## System Container Limitations

This section describes limitations for software running inside a system
container.

### Creating User Namespaces inside a System Container

System containers do not currently support creating a user-namespace
inside the system container and mounting procfs in it.

That is, executing the following instruction inside a system container
is not supported:

    unshare -U -i -m -n -p -u -f --mount-proc -r bash

The reason this is not yet supported is that Sysbox is not currently
capable of ensuring that the procfs mounted inside the unshared
namespace is the proper one. We expect to fix this soon.

## Sysbox Functional Limitations

### Sysbox must run as root on the host

Sysbox must run with root privileges on the host system. It won't
work if executed without root privileges.

Root privileges are necessary in order for Sysbox to interact with the Linux
kernel in order to create the containers and perform many of the advanced
functions it provides (e.g., procfs virtualization, sysfs virtualization, etc.)

### Checkpoint and Restore Support

Sysbox does not currently support checkpoint and restore of system containers.

### Sysbox Nesting

Sysbox must run at the host level (or within a privileged container if you must).

Sysbox does not work when running inside a system container. This implies that
we don't support running a system container inside a system container at this
time.
