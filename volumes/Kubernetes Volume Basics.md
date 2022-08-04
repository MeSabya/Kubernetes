# Kubernetes Volume Basics
Kubernetes supports many types of volumes. A Pod can use any number of volume types simultaneously. 

👉 *Ephemeral volume types have a lifetime of a pod, but persistent volumes exist beyond the lifetime of a pod. 
When a pod ceases to exist, Kubernetes destroys ephemeral volumes; however, 
Kubernetes does not destroy persistent volumes* 

👉 *For any kind of volume in a given pod, data is preserved across container restarts.*

👉 *To use a volume, specify the volumes to provide for the Pod in .spec.volumes and declare where to mount those volumes 
into containers in .spec.containers[*].volumeMounts.*

## Filesystem vs Volume vs Persistent Volume
In Kubernetes, each container can read and write to its own, isolated filesystem.

But, data on that filesystem will be destroyed when the container is restarted.

👉 To solve this, Kubernetes has volumes. Volumes let your pod write to a filesystem that exists as long as the pod exists.

Volumes also let you share data between containers in the same pod.

But, data in that volume will be destroyed when the pod is restarted.

👉 To solve this, Kubernetes has persistent volumes. Persistent volumes are long-term storage in your Kubernetes cluster.

Persistent volumes exist beyond containers, pods, and nodes.

A pod uses a persistent volume claim to to get read and write access to the persistent volume.

![image](https://user-images.githubusercontent.com/33947539/179805446-0d0f6b66-703b-4772-95c0-4aa7c0d24b3d.png)

### Example YAML for using a volume
There are two steps for using a volume.

- **First, the pod defines the volume.**
- **Second, the container uses volumeMounts to add that volume at a specific path (mountPath) in its filesystem.**

![image](https://user-images.githubusercontent.com/33947539/180333673-7991d991-ecbd-438e-9a0d-e40357baa9bf.png)


## Ephemeral Volumes:
- The emptyDir ephemeral volume
- Generic Ephemeral Volume
- Hostpath Volume  

There are a variety of use cases for ephemeral volumes including:

👉 Sharing data between containers in a Pod (multi-container Pods)

👉Providing read-only input configuration data to a Pod (Configmaps and secrets)

👉Temporary data caches

### The emptyDir ephemeral volume:
EmptyDir are volumes that get created empty when a Pod is created.

While a Pod is running its emptyDir exists. If a container in a Pod crashes the emptyDir content is unaffected. Deleting a Pod deletes all its emptyDirs.

There are several ways a Pod can be deleted. Accidental and deliberate. All result in immediate emptyDir deletion. emptyDir are meant for temporary working disk space.

#### Basic emptyDir Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myvolumes-pod
spec:
  containers:
  - image: alpine
    imagePullPolicy: IfNotPresent
    name: myvolumes-container
    
    command: [    'sh', '-c', 'echo The Bench Container 1 is Running ; sleep 3600']
    
    volumeMounts:
    - mountPath: /demo
      name: demo-volume
  volumes:
  - name: demo-volume
    emptyDir: {}
```
From Pod spec above:

```yaml
  volumes:
  - name: demo-volume
    emptyDir: {}
```
We define an emptyDir volume named demo-volume. The {} at the end means we do not supply any further requirements for the emptyDir .

This emptyDir spec makes it available to all containers in the Pod.

From Pod spec above:

```yaml
    volumeMounts:
    - mountPath: /demo
      name: demo-volume
```
Every container in the Pod needs to specify where it wants to have the emptyDir mounted.

Our example mounts the emptyDir at the mountPath: /demo

The name: demo-volume must refer to the volume at the bottom of the Pod spec. It specifies : mount demo-volume at /demo in the container.

### Usecase:
In my project there is a python component (deployed as a statefulset) which consists of 4 containers , namely : MDW, OS-Pack, RKT, MAG. 
MAG, RKT conatiner needs a bunch of static configuration files/cache files to execute a proper regression test case. Configmap cannot be used as it will be a lot of files. 

#### To Solve this issue:
We created a dedicated docker container with these configuration files/cache files and push them to shared openshift volumes so that containers can read them
So we introduced another container called GMC-Cache. Basically this container has all the cache files loaded at /projects/cache path.

Basically the flow goes like this :
MAG and RKT needs cache files. Cache files are present in another container gmc-cache. Now two questions arise:

##### 1. How gmc-cache get those cache files at /projects/cache path? 

We need two things here:
1. To create the container we need to build the image.
2. We need to add the cache files at /projects/cache path of the container.

Obviously we need a Dockerfile to build the container image. Then we need a shell script which must run when the container starts and that script should be responsible to 
copy the cache files to /projects/cache path.

**Dockerfile**

```Dockerfile
FROM dockerhub.rnd.net:5002/acs/rhel:8

USER root

RUN chmod -R 777 /opt/
RUN chown -R 1000 /opt/

#LSS cache files
RUN mkdir -p /cacheFiles/lss
COPY lss/* /cacheFiles/lss/

#ABR
RUN mkdir -p /cacheFiles/abr/
COPY abr/* /cacheFiles/abr/

#AAF


#AMA RFD
RUN mkdir -p /cacheFiles/ama_rfd/
COPY rfd/ama_rfd/* /cacheFiles/ama_rfd/

#Rail RFD
RUN mkdir -p /cacheFiles/rfd/
COPY rfd/RFD/* /cacheFiles/rfd/

#EZT
RUN mkdir -p /cacheFiles/ezt/
COPY EZT/* /cacheFiles/ezt/



COPY configure.sh /configure.sh
RUN chmod +x /configure.sh

#USER 1000

ENTRYPOINT ["/configure.sh"]
```

**configure.sh**

```shell
#!/bin/bash

# Enables job control
set -m

# Enables error propagation
set -e

# Echo with
log() {
  echo "[$(date +"%T")] $@"
}

log "Creating folders"
mkdir -p /projects/rdpdelde/abr
mkdir -p /projects/rdpdelde/AMA_RFD_SQLITE
mkdir -p /projects/rdpdelde/RFD_SQLITE
mkdir -p /projects/rdpdelde/MIGRATION_SQLITE
mkdir -p /projects/rdpdelde/LSS_EZT

log "Copy ABR mocked data"
cp "/cacheFiles/abr/"* /projects/rdpdelde/abr/
log "Copy RFD mocked data"
cp "/cacheFiles/rfd/"* /projects/rdpdelde/RFD_SQLITE/
log "Copy AMA RFD mocked data"
cp "/cacheFiles/ama_rfd/"* /projects/rdpdelde/AMA_RFD_SQLITE/
log "Copy EZT mocked data"
cp "/cacheFiles/ezt/"* /projects/rdpdelde/MIGRATION_SQLITE/
#log "Copy LSS mocked data"
#cp "/cacheFiles/lss/"* /projects/rdpdelde/LSS_EZT/



log "Maintaining the container running..."
tail -f /dev/null
```


##### 2. How RKT and MAG can access /projects/cache?

```yaml
- name: gmc-cache
  image: gmc-cache:v1.7 
  volumeMounts:
  - name: gmc-data
    mountPath: /projects/cache
    
volumes:
- name: gmc-data
  emptyDir: {}
```
Link gmc-data in the volumeMounts of your conatainer:

```yaml
   - name: rkt
   image: dockerfiles/rkt:18.0.37.0-1.0.6-15-47e65b3 
   ......
   volumeMounts: 
   .....
    - name: gmc-data
     mountPath: /projects/rdpdelde
```

To understand more on the emptyDir please follow the next section hostPath volume.

## HostPath Volume

A hostPath volume mounts a file or directory from the node's filesystem into the Pod. You can specify whether the file/directory must already exist on the node or should be created on pod startup. 
You can do it using a type attribute in the config file:

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: my-pod

spec:

  containers:

  - image: my-app-image

    name: my-app

    volumeMounts:

      - mountPath: /test-pd

        name: test-volume

  volumes:

  - name: test-volume 

    hostPath:

    path: /data  #directory on host

    type: Directory #optional
 ```

type: Directory defines that the directory must already exist on the host, so you will have to create it there manually first, before using the hostpath.

Other values for type are DirectoryOrCreate, File, FileOrCreate. Where *OrCreate will be created dynamically if it doesn't already exist on the host.

### Disadvantages:
A hostPath volume for StatefulSet should only be used in a single-node cluster, e.g. for development. Rescheduling of the pod will not work.

Instead, consider using a Local Persistent Volume for this kind of use cases.

The biggest difference is that the Kubernetes scheduler understands which node a Local Persistent Volume belongs to. With HostPath volumes, a pod referencing a HostPath volume may be moved by the scheduler to a different node resulting in data loss. 
But with Local Persistent Volumes, the Kubernetes scheduler ensures that a pod using a Local Persistent Volume is always scheduled to the same node.


## Persistent Volumes (PVs)

#### Defination:
A persistent volume (PV) is a cluster-wide resource that you can use to store data in a way that it persists beyond the lifetime of a pod. The PV is not backed by locally-attached storage on a worker node but by networked storage system such as EBS or NFS or a distributed filesystem like Ceph.

### When To Use Persistent Volumes?

You should use a persistent volume whenever you have data that needs to outlive individual pods. Unless the data is transitory or specific to a single container, it’s usually best stored in a persistent volume.

#### Here are some common use cases:

**Database storage**: Data in a database should always be stored in a persistent volume so it persists beyond the containers that run the server. You don’t want to wipe your users’ data each time the pod restarts.

**Log storage**: Writing container log files to a persistent volume ensures they’ll be available after a crash or termination. If they’re not written to a persistent volume, the crash will destroy the logs that could have helped you debug the issue.

**Protection of important data**: Persistent volumes let you avoid accidental data deletion. They include safeguards that prohibit the removal of volumes that are actively used by pods.

**Data independent of pods**: Persistent volumes make sense whenever your data is of primary importance in your cluster. They give you the tools to manage data independently of application containers, making it easier to handle backups, performance, and storage capacity allocations.




