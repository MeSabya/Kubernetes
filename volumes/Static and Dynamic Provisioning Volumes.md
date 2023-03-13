## Static provisioning volumes
In the previous lectures we discussed,  about how to create PVs.  And then create PVCs to claim that storage.  
And then use the PVCs,  in the pod definition files as volumes.  In this case we create a PVC,  from a Google Cloud persistent disk.  
The problem here is that,  before this PV is created, you must have created,  the disk on Google Cloud.  Every time an application requires storage,  
you have to first manually provision,  the disk on Google Cloud and then,  manually create a persistent volume definition file.  
Using the same name as that of the disk that you created.  That's called **static provisioning volumes**.

It would've been nice, if the volume,  gets provisioned automatically,  when the application requires it.  
And that's where storage classes come in.  With storage classes you can define,  a provisioner such as Google Storage.  
That can automatically provision storage  on Google Cloud and attach that to pods,  when a claim is made.  
That's called **dynamic provisioning of volumes**.  

You do that by creating a **storage class object**.



