## Cluster Setup and Hardening
   1. Authentication
   2. Authorization
   3. Service Accounts
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

## System Hardening
   1. Minimize OS footprint
   2. Limited Node access
   3. SSH hardening
   4. [Privilege escalation in linux](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/001.%20System%20Hardening/00.%20Privilege%20Escalation%20in%20Linux.md)
   5. Remove obsolete packages and services
   6. Restrict kernel modules
   7. Identify and disable open ports
   8. minimize IAM roles
   9. UFW firewall basics
   10. Restrict syscalls using seccomp
   11. [Seccomp in k8s](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/001.%20System%20Hardening/01.%20Tracing%20and%20Restricting%20Syscalls.md)
   12. [Apparmor](https://github.com/MeSabya/Kubernetes/blob/main/01.%20CKS/001.%20System%20Hardening/02.%20AppArmor.md)

## Minimize Microservices Vulnerabilities
   1. Admission Controller
   2. Open Policy Agent

## Supply Chain security 
   1. Minimize base image foorprint
   2. Image security
   3. Secure your supply chain
   4. use static analysis of workloads
   5. scan images for known vulenrabilities

## Monitoring , Logging
  1. Detect Malicious activity
  2. Detect threats
  3. Detect all phases of attacks
  4. Perform deep analytical investigation
  5. Immutability of containers
  6. use audit logs to monitor access
