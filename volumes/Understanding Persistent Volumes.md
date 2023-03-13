## Understanding Persistent Volumes

### Problem Statement: 
When we created volumes in the previous section  we configured volumes within the pod definition file  so every configuration information  required to configure storage for the volume  goes within the pod definition file.  Now, when you have a large environment  with a lot of users deploying a lot of pods  the users would have to configure storage  every time, for each pod.  Whatever storage solution is used  the users who deploys the pods  would have to configure that  on all pod definition files in his environment.  Every time a change is to be made  the user would have to make them on all of his pods.  Instead, you would like to manage storage more centrally.  You would like it to be configured in a way  that an administrator can create a large pool of storage  and then have users carve out pieces from it, as required.  That is where persistent volumes can help us.  

👉 **A persistent volume is a cluster wide  pool of storage volumes,  configured by an administrator to be used by users deploying applications on the cluster.**

The fact that we have a few EBS volumes available does not mean that Kubernetes knows about their existence. We need to add PersistentVolumes that will act as a bridge between our Kubernetes cluster and AWS EBS volumes.

PersistentVolumes allow us to abstract details of how storage is provided (e.g., EBS) from how it is consumed. Just like Volumes, PersistentVolumes are resources in a Kubernetes cluster. 
The main difference is that their lifecycle is independent of individual Pods that are using them.

### Looking into the Definition

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: manual-ebs-01
  labels:
    type: ebs
spec:
  storageClassName: manual-ebs
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: REPLACE_ME_1
    fsType: ext4
...
```

## Usage of Persistent Volumes
Kubernetes persistent volumes are useless if no one uses them. They exist only as objects with relation to, in our case, specific EBS volumes. 
They are waiting for someone to claim them through the PersistentVolumeClaim resource.

Just like Pods which can request specific resources like memory and CPU, PersistentVolumeClaims can request particular sizes and access modes.

### How to Claim Persistent Volumes?

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins
  namespace: jenkins
spec:
  storageClassName: manual-ebs
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
The YAML file defines a PersistentVolumeClaim with the storage class name manual-ebs. That is the same class as the persistent volumes manual-ebs-* we created earlier. The access mode and the storage request are also matching what we defined for the persistent volume.

👉 Please note that we are not specifying which volume we’d like to use. Instead, this claim specifies a set of attributes (storageClassName, accessModes, and storage). Any of the volumes in the system that match those specifications might be claimed by the PersistentVolumeClaim named jenkins.

👉 Bear in mind that resources do not have to be the exact match. Any volume that has the same or bigger amount of storage is considered a match. A claim for 1Gi can be translated to at least 1Gi. In our case, a claim for 1Gi matches all three persistent volumes since they are set to 5Gi.

## Creating Deployment for Attaching Claimed Volumes to Pods

cat pv/jenkins-pv.yml

```yaml
...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins
spec:
  ...
  template:
    ...
    spec:
      containers:
      - name: jenkins
        ...
        volumeMounts:
        - name: jenkins-home
          mountPath: /var/jenkins_home
        ...
      volumes:
      - name: jenkins-home
        persistentVolumeClaim:
          claimName: jenkins
      ...
 ```
## Verification 
We’ll wait until the Deployment rolls out before proceeding with a test that will confirm whether Jenkins state is now persisted.

```shell
kubectl --namespace jenkins \
    rollout status \
    deployment jenkins
```

Once the rollout is finished, we’ll see a message stating that the deployment "jenkins" was successfully rolled out.

- We sent a request to the Kubernetes API to create a Deployment. 
- As a result, we got a ReplicaSet that, in turn, created the jenkins Pod. 
- It mounted the PersistentVolumeClaim, which is bound to the PersistenceVolume, that is tied to the EBS volume. 
- As a result, the EBS volume was mounted to the jenkins container running in a Pod. 
 

