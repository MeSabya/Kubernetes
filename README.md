 # Kubernetes Basics Table of Content:
 1. Kubernetes Architecture in detail
 2. Kubernetes controllers
   - Replicaset 
   - Deployment 
   - Statefulset 
   - jobs 
3. Volumes 
4. Services (Network)
5. HealthChecks 
6. Configuration (Configmaps and secrets).
7. Memory and CPU constraint.
8. Microservice Design Pattern.(init, sidecar)
9. Pod Placement and schedulling. 
10. Security in K8

![image](https://user-images.githubusercontent.com/33947539/192934688-1b8bfce5-4a07-4f65-8662-5f68428e917c.png)

Glossary:- 
=========
- Images
- Pods
- Labels and Selectors
- Replication Controller
- Services
- Volumes
- Persistent Volumes
- Secrets
- Names
- Namespaces
- Nodes
- Security Context
- Service Accounts
- Annotations
- Daemon Sets
- Deployments
- Ingress Resources
- Horizontal Pod Autoscaling
- Jobs
- Resource Quotas
- Replica Sets
 
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
 - [Practice Questions-2](https://github.com/bmuschko/ckad-crash-course)
 - [ ] [Practice Questions-1](https://medium.com/bb-tutorials-and-thoughts/practice-enough-with-these-questions-for-the-ckad-exam-2f42d1228552)
 - [ ] [ CKAD Exercises](https://github.com/dgkanatsios/CKAD-exercises)
 - [ ] [Network Policy in K8s](https://github.com/ahmetb/kubernetes-network-policy-recipes)
 - [ ] [Practice Challnges](https://www.katacoda.com/liptanbiswas/courses/ckad-practice-challenges)
 - [ ] [K8 exam simulator](https://killer.sh/ckad)
 - [ ] [Material](https://kodekloud.com/courses/certified-kubernetes-application-developer-ckad/)
 - [ ] [CKAD Syllabus](https://github.com/cncf/curriculum/blob/master/CKAD_Curriculum_v1.22.pdf)
 - [ ] [ckad-prep-notes](https://github.com/twajr/ckad-prep-notes)
 - [ ] [ckad-practice-questions](https://dev.to/liptanbiswas/ckad-practice-questions-4mpn)
 - [ ] [Practice Questions](https://www.katacoda.com/courses/kubernetes)
 - [ ] [Question Banks](https://luafanti.medium.com/certified-kubernetes-application-developer-ckad-everything-you-need-to-know-30eb5c2f70ba)
 - [ ] [CKAD Resources](https://github.com/lucassha/CKAD-resources)
 [K8 CheetSheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
 
# Links I find interesting to CKAD topic
https://codeburst.io/kubernetes-ckad-hands-on-challenge-13-replicaset-without-downtime-a9468d1fa994

https://codeburst.io/kubernetes-ckad-weekly-challenge-7-migrate-a-service-68c7af41c8df

https://codeburst.io/kubernetes-ckad-weekly-challenges-overview-and-tips-7282b36a2681

https://codeburst.io/the-ckad-browser-terminal-10fab2e8122e

https://en.sokube.ch/post/kubernetes-tips-for-your-ckad-exam-by-cncf

https://github.com/AmundsenJunior/yet-another-ckad-training-resource/blob/master/Udemy-CKAD-Practice.md

https://github.com/dgkanatsios/CKAD-exercises

https://github.com/topics/ckad-exercises

https://itnext.io/practical-tips-for-passing-ckad-exam-6cbdf2d35cb1

https://matthewpalmer.net/kubernetes-app-developer/#purchase-the-ebook

https://matthewpalmer.net/kubernetes-app-developer/articles/ckad-practice-exam.html

https://medium.com/@milan.das77/how-to-pass-ckad-cka-on-your-first-attempt-1c19e6c20335

https://medium.com/bb-tutorials-and-thoughts/practice-enough-with-these-questions-for-the-ckad-exam-2f42d1228552

https://medium.com/faun/kubernetes-ckad-weekly-challenge-2-namespaces-deployments-and-services-de1ede24679a

https://www.katacoda.com/ckad-prep/scenarios/first-steps

https://www.katacoda.com/fabito/scenarios/ckad

https://www.katacoda.com/fabito/scenarios/ckad

https://www.reddit.com/r/kubernetes/comments/kl1h4m/ckad_exam_laptop_setup_query/
