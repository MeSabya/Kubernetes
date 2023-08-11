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


