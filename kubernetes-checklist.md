# Kubernetes Pod Security Checklist

## Network Policies

Network policies act similar to firewalls in Kubernetes cluster, i.e. they define what network access a pod should have at the ingress (incoming connections) and egress (outgoing connections) level.

A pod should only be able to access (and be accessed by) resources it truly needs.

Recommendation:

* Start by doing a general deny all for ingress and egress for all pods in a namespace.
* Then add permissions for each pod/set of pods


## No root containers

A container should never (with the exception of very rare cases) be run by root. They should also have a minimal set of capabilities. A good starting point may be:

```
securityContext:
  runAsNonRoot: true  # do not allow the container to run as root
  allowPrivilegeEscalation: false  # do not allow the container to switch to a user with more permissions
  privileged: false  # the container is not privileged (usually the default)
  runAsUser: <uid/username>  # run as user
  runAsGroup: <gid/group name>  # run as group
  capabilities:  # drop all extra capabilities
    drop:
      - all
```

## Use Secrets for sensitive data

Any sensitive information should be saved in secrets, not be hardcoded or saved in configmaps.


## Resource limits

Make sure to set resource limits (cpu, ram, ephemeral-storage) for all pods to prevent them from using to many resources on the node. `resources.limits` set the limits for the pod, while `resources.requests` is used for scheduling.

```
resources:
  limits:
    cpu: 200m
    memory: 128Mi
    ephemeral-storage: 300Mi
  requests:
    cpu: 50m
    memory: 32Mi
    ephemeral-storage: 100Mi
```

## Service accounts

Do not use the default service accounts, and do not give any permissions to them using RBAC. Create new service accounts for each deployment.


## Make containers immutable

It is much more difficult to modify a container if the entire file system is read-only. You may need to mount some `emptyDir` volumes to give `read-write` access in certain folders for your service to function.

```
securityContext:
  runAsNonRoot: true
```

## Limit the syscalls from the containers

You can use SecComp, AppArmor, and SELinux to limit the permitted syscalls for the containers. Remember that they must be installed on the host node in order to have any effect.

```
securityContext:
  seccompProfile:
    type: RuntimeDefault
```

```
securityContext:
  seLinuxOptions:
    level: "s0:c123,c456"
```

```
annotations:
  container.apparmor.security.beta.kubernetes.io/<container name>: runtime/default
```
