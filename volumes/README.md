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

We can create our application deployment using this PVC.
![image](https://user-images.githubusercontent.com/33947539/139820403-af477ed9-cb8d-4dd2-93bc-b8021e4441c0.png)![image](https://user-images.githubusercontent.com/33947539/139820612-a0b016cb-bdb4-49a6-852d-4ebd21497beb.png)

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





