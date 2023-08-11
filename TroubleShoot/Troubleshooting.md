## How to debug controlplane failure
Step1: Check nodes.

Step2 : check pods if they are running healthy.

Step3: Check controlplane pods in kube-system ns.

Step4: Or else check the controlplane services. 

       service kube-apiserver status 
	   
	   service kube-controller-manager status 
	   
	   service kube-scheduler status 
	   
	   service kubelet status 
	   
	   service kube-proxy status 
	   
Step5: Check the logs of the control plane components 
       
	   kubectl logs kube-apiserver-master -n kube-system 
	   
Step6: Check the logs of the services using hosts logging mechanism 
       
		sudo journalctl -u kube-apiserver 



## How to debug worker node failure
![image](https://github.com/MeSabya/Kubernetes/assets/33947539/98558998-d72b-46e8-be31-75ffd426efe5)

### Scenario 1

**node01 is not ready, fix the same**

```
kubectl get nodes

NAME           STATUS     ROLES           AGE     VERSION
controlplane   Ready      control-plane   5m43s   v1.26.0
node01         NotReady   <none>          5m7s    v1.26.0
```

**Step 1**: SSH to node01 and check the status of the container runtime (containerd, in this case) and the kubelet service.

```
root@node01:~> systemctl status containerd
● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-12-29 14:50:26 EST; 11min ago
       Docs: https://containerd.io
   Main PID: 1058 (containerd)
      Tasks: 65
     Memory: 181.2M
     CGroup: /system.slice/containerd.service

root@node01:~>

root@node01:~> systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead) since Thu 2022-12-29 14:51:58 EST; 7min ago
       Docs: https://kubernetes.io/docs/home/
    Process: 2085 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTR>
   Main PID: 2085 (code=exited, status=0/SUCCESS)
```

Since the kubelet is not running, attempt to start it by running the following command:

```
root@node01:~> systemctl start kubelet

root@node01:~> systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Tue 2023-05-30 13:06:47 EDT; 6s ago
       Docs: https://kubernetes.io/docs/home/
   Main PID: 4313 (kubelet)
      Tasks: 15 (limit: 77091)
     Memory: 31.4M
     CGroup: /system.slice/kubelet.service
```

node01 should go back to ready state now.
