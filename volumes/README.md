# Volumes
Kubernetes volumes may be classified into: 
- **host-based storage**:Similar to Docker volumes, where a portion of the host’s storage becomes available to the pod. Once a pod is terminated, the volume gets automatically deleted.
- **non-host-based storage**: It doesn’t rely on a specific node. Instead, a storage volume is created from an external storage service. Volumes based on this storage type would be available even after the pods are deleted.

## Details on host-based storage:
Any storage that the containers use inside the pod is ephemeral by nature. They get removed automatically with every pod restart.
Two specific volume types that are dependent on host-based storage are emptyDir and and hostPath.
### EmptyDir
- An emptyDir volume is first created when a Pod is assigned to a Node and initially its empty
- A Volume of type emptyDir that lasts for the life of the Pod, even if the Container terminates and restarts.
- If a container in a Pod crashes the emptyDir content is unaffected.
- All containers in a Pod share use of the emptyDir volume .
- Each container can independently mount the emptyDir at the same / or different path.
- Using emptyDir, The Kubelet will create the directory in the container, but not mount any storage.
- When a Pod is removed from a node for any reason, the data in the emptyDir is deleted forever along with the container.
- The location should of emptyDir should be in /var/lib/kubelet/pods/{podid}/volumes/kubernetes.io~empty-dir/ on the given node where your pod is running.

**Some of the popular use cases for using emptyDir volumes include:**
- The creation of a scratch disk for storing intermediary data.
- A common storage area for sharing configuration settings and metadata across multiple containers of the same pod.
- A well-known storage location for containers to store and forward data. A crawler container might populate the volume periodically, while the web server is responding to the 
  requests.
  
### HostPath
- A hostPath volume mounts a file or directory from the node's filesystem into the Pod. 
- You should NOT use hostPath volume type for StatefulSets

You can specify whether the file/directory must already exist on the node or should be created on pod startup. 
You can do it using a type attribute in the config file:

![image](https://user-images.githubusercontent.com/33947539/139912040-94363d69-b2fb-4160-9ad4-527c04000e09.png)

**type**: Directory defines that the directory must already exist on the host, so you will have to create it there manually first, before using the hostpath.

Other values for type are DirectoryOrCreate, File, FileOrCreate. Where *OrCreate will be created dynamically if it doesn't already exist on the host.



**Some uses for a hostPath are:**

- Running a Container that needs access to Docker internals; use a hostPath of /var/lib/docker
- Running cAdvisor in a Container; use a hostPath of /sys

## Details on non-host-based storage:

# Persistent Volumes
- Kubernetes makes physical storage devices like your SSDs, NVMe disks, NAS, NFS servers available to your cluster in the form of objects called -- Persistent Volumes.
- If you’re using Kubernetes on Google’s or Amazon’s cloud, you can have your google SSDs or EBS volumes available to your containers in the form of persistent volumes.
- Each of these Persistent Volumes is consumed by a Kubernetes Pod (or Pods) by issuing a PersistentVolumeClaim object -- a PVC. 
- **A PVC object lets pods use storage from Persistent Volumes.**
- A Persistent Volume is an abstraction for the physical storage device that you have attached to the cluster. Your pods can use this storage space using Persistent Volume Claims. The easiest way to create the PV/PVC pair for your Pod is to use a **StorageClass object**, and then using the storageclass to create your PV-PVC pair dynamically whenever you need to use it.

![image](https://user-images.githubusercontent.com/33947539/139818756-d2f3c0a5-280f-4ef4-a0dc-a4083aa9f5bc.png)

# Persistent Volume Claim
A persistent volume claim is a dedicated storage that kubernetes has carved out for your application pod, from the storage that was made available using the storage class.
Let’s create a PVC yaml using the storage class…

![image](https://user-images.githubusercontent.com/33947539/139819885-ca6257e5-2c16-400d-bd4a-b25f4dc036c1.png)

In this case, we are telling kubernetes to use the storage class ‘local-device’ to create a Persistent Volume with 5Gi of storage capacity and RWO access mode.

We can create this resource and check for the following outputs:

**kubectl get pv**

NAME                                       
pvc-079bbc07-e2fb-412a-837b-4745051c1bfc
and

**kubectl get pvc**

NAME                                           STATUS                                VOLUME                                    
local-device-pvc                               Bound                pvc-079bbc07-e2fb-412a-837b-4745051c1bfc

### We can create our application deployment using this PVC:




![image](https://user-images.githubusercontent.com/33947539/139820403-af477ed9-cb8d-4dd2-93bc-b8021e4441c0.png)
![image](https://user-images.githubusercontent.com/33947539/139820612-a0b016cb-bdb4-49a6-852d-4ebd21497beb.png)


Following these operations, the Kubelet will mount a volume that matches the specifications of the PVC to the application container.

## What is the difference between persistent volume (PV) and persistent volume claim (PVC) in simple terms?
So a persistent volume (PV) is the "physical" volume on the host machine that stores your persistent data. A persistent volume claim (PVC) is a request for the platform to create a PV for you, and you attach PVs to your pods via a PVC.

Something akin to

Pod -> PVC -> PV -> Host machine

## Details On Persistent Volume
Persistent volume consist of storage capacity, volume type, reclaim policy and access modes.

**Storage Capacity**: It’s define the storage space allow for volume in the cluster by the administrator.

**Volume Type**: It’s define which type of volume you want to use e.g emptyDir, hostPath, gitRepo, azureDisk etc. We’ll be using one of them in our session.
- emptyDir — A simple empty directory used for storing transient data.
- hostPath — Used for mounting directories from the worker node’s filesystem into the pod.

**Reclaim Policy**: It’s define what to do with the volume after the bound is broken from the persistent volume claim. 

 **There are three policies**: 
 1. Delete: PersistentVolume will be deleted when the PVC is deleted but data will persist. 
 2. Recycle: Volume’s contents will be deleted Persistent Volume will be available to be claimed again. 
 3. Retain: It is default policy,Kubernetes will retain the volume and its contents after it’s released from its claim to make PersistentVolume available again for claims can be 
    done by delete and recreate the PersistentVolume resource manually Underlying storage can either delete or left to be reused 
    by the next pod.

**Access Modes**: There are three of them: 
- RWO — ReadWriteOnce: Only a single node can mount the volume for reading and writing. 
- ROX — ReadOnlyMany: Multiple nodes can mount the volume for reading. 
- RWX — ReadWriteMany : Multiple nodes can mount the volume for both reading and writing.

## Details On Persistent Volume Claim

Persistent volume claim consist of storage class, resources and access modes.

**Storage Class**: The StorageClass resource specifies which provisioner should be used for provisioning the PersistentVolume when a PersistentVolumeClaim requests this StorageClass.

**Resources**: It’s defines that at which volumes this claim should trigger. For example if you write 500M in resource it will trigger to all Persistent volumes having storage of 500 Mb.

Access Modes are same as in persistent volume.

![image](https://user-images.githubusercontent.com/33947539/139830373-ad227725-5ec0-44d4-837c-5cd7202e1372.png)





