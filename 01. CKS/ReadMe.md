## Cluster Setup and Hardening
   1. Authentication
   2. Authorization
   3. [Service Accounts](https://github.com/MeSabya/Kubernetes/blob/main/SecurityInK8s/ServiceAccount.md)
      - [How Does Kubernetes Service Account Works?]()
      - [What is authorization token in Sservice Account]()
      - [Along with the service account, why role and role binding was needed?]()
   4. TLS in K8s
       - [How Asymmetric encryption works](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/01.%20TLS/00.%20TLS%20Basics.md#how-asymmetric-encryption-works)
       - [TLS and SSH relation](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/01.%20TLS/00.%20TLS%20Basics.md#tls-and-ssh-asymmetric-encryption-both-are-related)
       - [CA in k8s](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/01.%20TLS/01.%20Certificate%20Authority.md#certificate-authority-in-k8-cluster)
       - [How TLS work with ca.crt](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/01.%20TLS/01.%20Certificate%20Authority.md#how-tls-work-with-these-cakey--cacrt)
       - [TLS in K8s](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/01.%20TLS/01.%20Certificate%20Authority.md#tls-in-kubernetes)
       - [Certificate Details](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/01.%20TLS/02.%20Certificate%20Generation%20and%20View.md)
       - [How a POD authenticate with Api Server](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/01.%20TLS/03.%20How%20Pod%20authenticates%20with%20Api%20server.md)
   5. Protect node metadata and endpoint
   6. Securing k8s dashboard
   7. Verifying platform binaries
   8. [Upgrade k8s cluster](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/03.%20Cluster%20upgrade%20Process/00.Basics.md)
   9. [Network policies](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/02.%20Network%20Policy/00.Basics%20Of%20Network%20Policy.md)
   10. Securing ingress
   11. [Two different ways to start the Docker daemon](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/04.%20Docker%20in%20foreground%20and%20background.md)
   12. [Securing Docker Service using TLS](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/05.%20Securing%20Docker%20Service%20using%20TLS.md)
   13. [Secret in K8s](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/000.%20Cluster%20Setup%20And%20Hardening/06.Secrets.md)

## System Hardening
   1. Minimize OS footprint
   2. Limited Node access
   3. SSH hardening
   4. [Privilege escalation in linux](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/001.%20System%20Hardening/00.%20Privilege%20Escalation%20in%20Linux.md)
   5. Remove obsolete packages and services
   6. Restrict kernel modules
   7. [Identify and disable open ports](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/001.%20System%20Hardening/03.%20Disable%20Open%20Ports.md)
   8.  minimize IAM roles
   9.  [UFW firewall basics](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/001.%20System%20Hardening/05.%20UFW.md)
   10. Restrict syscalls using seccomp
   11. [Seccomp in k8s](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/001.%20System%20Hardening/01.%20Tracing%20and%20Restricting%20Syscalls.md)
   12. [Apparmor](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/001.%20System%20Hardening/02.%20AppArmor.md)
   13. [seccomp Vs AppArmor](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/001.%20System%20Hardening/04.%20Seccomp%20Vs%20Apparmor.md)
   14. 

## Minimize Microservices Vulnerabilities
   1. [Admission Controller](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/002.%20MINIMIZE%20MICROSERVICE%20VULNERABILITIES/001.Admission%20Controllers.md)
   2. [Open Policy Agent](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/002.%20MINIMIZE%20MICROSERVICE%20VULNERABILITIES/002.%20Open%20Policy%20Agent/00.%20Basics%20of%20OPA.md)

## Supply Chain security 
   1. [Minimize base image foorprint](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/003.%20Supply%20Chain%20Security/000.%20Minimize%20Base%20Image%20Footprint.md)
   2. Image security
   3. Secure your supply chain
   4. [use static analysis of workloads](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/003.%20Supply%20Chain%20Security/002.%20Static%20Analysis%20of%20k8s%20manifests.md)
   5. [scan images for known vulenrabilities](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/003.%20Supply%20Chain%20Security/003.%20Scan%20Images%20for%20Vulnerabilities.md)
   6. [what are ways we can restrict users to use image from Whitelist Allowed Registries](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/003.%20Supply%20Chain%20Security/001.%20Whitelist%20allowed%20registries.md)

## Monitoring , Logging
  1. Detect Malicious activity
  2. Detect threats
  3. Detect all phases of attacks
  4. Perform deep analytical investigation
  5. Immutability of containers
  6. use audit logs to monitor access
