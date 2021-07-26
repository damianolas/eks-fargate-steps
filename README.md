# Premessa
Risorse create con questa procedura:
- AWS EKS cluster Fargate
- AWS Network Load Balancer (internet facing)

# Procedura
1. Crea VPC (DNS hostname -> Enable), subnet e route tables
2. ECR: crea VPC endpoint per "dkr" e "api"  
3. Crea cluster EKS (Subnet pubbliche per ControlPlane e private per i pods)
4. Pod execution role: https://docs.aws.amazon.com/eks/latest/userguide/fargate-getting-started.html#fargate-sg-pod-execution-role  
5. Fargate profile: https://docs.aws.amazon.com/eks/latest/userguide/fargate-getting-started.html#fargate-gs-create-profile  
6. Crea un Fargate Profile per "kube-system" (scegli subnet private)
7. Configura il terminal su Mac (aws CLI, access keys)  
8. Aggiungi al terminal il cluster creato: ``` aws eks --region <value> update-kubeconfig --name <cluster_name> --profile <aws_profile> ```  
10. Se sono pending, fai ripartire i pod di CoreDNS: ``` kubectl rollout restart -n <kube-system> deployment <coredns> ``` 
12. Crea un Fargate Profile per il namespace custom dedicato ai pod di progetto
17. Logging con Fluent: (https://docs.aws.amazon.com/eks/latest/userguide/fargate-logging.html) + policy via AWS CLI   
14. Bilanciatore: https://docs.aws.amazon.com/eks/latest/userguide/load-balancing.html  
15. IAM OIDC provider: https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html 
17. Tagga le subnet pubbliche: ``` kubernetes.io/role/elb = 1 ``` se non viene fatto in automatico (questo è il caso di un ELB internet-facing)
18. Crea controller dei bilanciatori (caso target IP e non server) https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html (usa helm)
19. Crea un Fargate Profile per il namespace "ingress-nginx" 
20. Installa nginx:  
	caso NLB: 
    ``` wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/aws/deploy.yaml ```  
    Cancella riga Allow Priviledges e imposta le annotation opportune per il bilanciatore
    ``` Kubectl apply -f deploy.yaml ```  
    ``` kubectl edit svc ingress-nginx-controller -n ingress-nginx ```  
		A mano bisogna creare i configmaps tcp-services e udp-services  
		Nello yaml di nginx ``` - --tcp-services-configmap=ingress-nginx/tcp-services ```  
		Crea configMaps per tcp services (caso mqtt)  
15. Deploy Kube dashboard (prima crea il fargate profile per il namespace kubernetes-dashboard): https://docs.aws.amazon.com/eks/latest/userguide/dashboard-tutorial.html  
16. Crea ingress
18. Dischi e persistenza: (https://docs.aws.amazon.com/eks/latest/userguide/storage.html)  
19. Configura certificati su secret (se necessario) https://medium.com/faun/mount-ssl-certificates-in-kubernetes-pod-with-secret-8aca220896e6  
20. Allarmi CloudWatch se utili (come unhealthy host sul bilanciatore)  
21. Dare permessi a EKS (console e terminale) a chi non ha creato il cluster: https://aws.amazon.com/it/premiumsupport/knowledge-center/eks-api-server-unauthorized-error/
