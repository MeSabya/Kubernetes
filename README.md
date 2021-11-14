 ## Kubernetes Study Content
 1. [Deployments-vs-Statefulsets-vs-Daemonsets](https://github.com/MeSabya/Kubernetes/tree/main/counter-app#deployments-vs-statefulsets-vs-daemonsets)
    1. [Deployments](https://github.com/MeSabya/Kubernetes/tree/main/counter-app#deployments)
    2. [Statefulsets](https://github.com/MeSabya/Kubernetes/tree/main/counter-app#statefulsets)
       - [complexities-associated-with-replicaset-for-stateful-application](https://github.com/MeSabya/Kubernetes/tree/main/counter-app#complexities-associated-with-replicaset-for-stateful-application)
    3. [DaemonSets](https://github.com/MeSabya/Kubernetes/tree/main/counter-app#daemonsets)
    4. [Why StatefulSets? Can't a stateless Pod use persistent volumes?](https://github.com/MeSabya/Kubernetes/tree/main/counter-app#so-after-all-these-discussions-we-should-able-to-answer-the-question)
2. [microservice-design-pattern](https://github.com/MeSabya/Kubernetes/tree/main/microservice-design-pattern)
   1. [init Container Pattern](https://github.com/MeSabya/Kubernetes/tree/main/microservice-design-pattern/k8-init-container-pattern#init-container-pattern)
   2. [Sidecar Pattern](https://github.com/MeSabya/Kubernetes/tree/main/microservice-design-pattern/k8s-sidecar-container-pattern#microservice-architecture-sidecar-pattern)
3. [Understanding CPU Resources](https://github.com/MeSabya/Kubernetes/tree/main/ManagingResource#understanding-cpu-resources) 
   1. [Create container without resources limits](https://github.com/MeSabya/Kubernetes/tree/main/ManagingResource#create-container-without-resources-limits)
   2. [Create container with resources limits](https://github.com/MeSabya/Kubernetes/tree/main/ManagingResource#create-container-with-resources-limits)
   3. [cpu request exceeding available resources](https://github.com/MeSabya/Kubernetes/tree/main/ManagingResource#cpu-request-exceeding-available-resources)
4. [What is a headless service, what does it do/accomplish, and what are some legitimate use cases for it?](https://github.com/MeSabya/Kubernetes/blob/main/HeadlessService.md#what-is-a-headless-service-what-does-it-doaccomplish-and-what-are-some-legitimate-use-cases-for-it)
5. [Deployment in detail](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md)
   1. [Deployment Stratergies](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md#deployment-stratergies) 
      - [Rolling update](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md#rollingupdate-below)
      - [Rolling back](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md#rolling-back-to-previous-version)
   2. [minreadyseconds-affect-readiness-probe](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md#how-does-minreadyseconds-affect-readiness-probe)
   3. [Rollingupdate UseCase](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md#deployment-usecase-analysis)
   4. [Kubernetes Pod Affinity and Anti-Affinity](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md#kubernetes-pod-affinity-and-anti-affinity)
   5. [Deployment Commands](https://github.com/MeSabya/Kubernetes/blob/main/Deployments/Deployment.md#commands-used-in-deployment)
 
## References:
- https://github.com/vfarcic/k8s-specs.git
 
 ## CKAD Prepration material:
 - [ ] [Practice Questions] (https://medium.com/bb-tutorials-and-thoughts/practice-enough-with-these-questions-for-the-ckad-exam-2f42d1228552)
 - [ ] [ CKAD Exercises] (https://github.com/dgkanatsios/CKAD-exercises)
 - [ ] [Network Policy in K8s] (https://github.com/ahmetb/kubernetes-network-policy-recipes)
 - [ ] [Practice Challnges] (https://www.katacoda.com/liptanbiswas/courses/ckad-practice-challenges)
 - [ ] [K8 exam simulator] (https://killer.sh/ckad)
 - [ ] [Material] (https://kodekloud.com/courses/certified-kubernetes-application-developer-ckad/)
 - [ ] [CKAD Syllabus] (https://github.com/cncf/curriculum/blob/master/CKAD_Curriculum_v1.22.pdf)
 
 [K8 CheetSheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
