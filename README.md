  AWS INGRESS LOAD BALANCER SETUP   
  *****************************************************************************************************************************************************************
    
    
    ********** COMMANDS TO TROUBLESHOOT THE LOAD BALANCERS****************************************    
        - kubectl get po -n cert-manager
        - kubectl describe po -n cert-manager <cert-manager-pod-name>
        - kubectl get ing -A
        - kubectl describe ing/<your-ingress> -n <your-namespace>
        - ​kubectl get sa (In SA annotation need to be attached)
        - kubectl logs -n <kube-system>   deployment.apps/<aws-load-balancer-controller> 
        
    *********************************************************************************************************************
    ######################################## STEPS TO CREATE THE LOAD BALANCER WITH INGRESS #######################################
    
    
    1)  Create an IAM policy.
    
        curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.1/docs/install/iam_policy.json
        
    2)  aws iam create-policy \
        --policy-name AWSLoadBalancerControllerIAMPolicy \
        --policy-document file://iam_policy.json
        
        A) View your cluster's OIDC provider URL.
          
          aws eks describe-cluster --name hive-prod --query "cluster.identity.oidc.issuer" --output text
          
          Example output:
          
         https://oidc.eks.us-east-1.amazonaws.com/id/7A899DB907EDBE036CE17A894B740C1F
          
        B) Create the IAM role.
        
            aws iam create-role \
            --role-name AmazonEKSLoadBalancerControllerRole \
            --assume-role-policy-document file://"load-balancer-role-trust-policy.json"
            
        C)  Attach the required Amazon EKS managed IAM policy to the IAM role. 
        
        aws iam attach-role-policy \
        --policy-arn arn:aws:iam::626690110339:policy/AWSLoadBalancerControllerIAMPolicy \
        --role-name AmazonEKSLoadBalancerControllerRole
        
        D)Save the following contents to a file that's named aws-load-balancer-controller-service-account.yaml.
        
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          labels:
            app.kubernetes.io/component: controller
            app.kubernetes.io/name: aws-load-balancer-controller
        name: aws-load-balancer-controller
        namespace: kube-system
        annotations:
          eks.amazonaws.com/role-arn: arn:aws:iam::626690110339:role/AmazonEKSLoadBalancerControllerRole
          
        E)Create the Kubernetes service account on your cluster. The Kubernetes service account named 
        aws-load-balancer-controller is annotated with the IAM role that you created named AmazonEKSLoadBalancerControllerRole.
        
        kubectl apply -f aws-load-balancer-controller-service-account.yaml
        
    5)  Install the AWS Load Balancer Controller using Helm V3 or later or by applying a Kubernetes manifest.
    If you want to deploy the controller on Fargate, use the Helm procedure because it doesn't depend on cert-manager.

        A) Install cert-manager using one of the following methods to inject certificate configuration into the webhooks .
        
        kubectl apply \
        --validate=false \
        -f https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.yaml
        
        B)Install the controller.
        
       * curl -Lo v2_4_1_full.yaml https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.4.1/v2_4_1_full.yaml
         
       * sed -i.bak -e 's|your-cluster-name|hive-dev-eksctl|' ./v2_4_1_full.yaml 
         
       * Remove the service account in v2_4_1_full.yaml
       
       apiVersion: v1
       kind: ServiceAccount
       metadata:
         labels:
           app.kubernetes.io/component: controller
           app.kubernetes.io/name: aws-load-balancer-controller
       name: aws-load-balancer-controller
       namespace: kube-system
       
       * kubectl apply -f v2_4_1_full.yaml
       
       * kubectl get deployment -n kube-system aws-load-balancer-controller





    
          
          
        
        
        
        
        
        
        
        
        
