# How to Set Up EKS on AWS?

## Course Details

| Information  |                   |
|--------------|      :-----:      |
| Name    | Sebastian Guerra - Juan David Prieto - Juan David Echeverri - Santiago Puerta |
| Email   | jsguerrah@eafit.edu.co - jdprietom@eafit.edu.co - jdecheverv@eafit.edu.co - spuertaf@eafit.edu.co |
| Teacher | Edwin Montoya          |
| Course  | ST0263                 |

## Description 
This guide serves as an operational reference to establish your own managed Kubernetes environment on AWS.

The information and parameters of the project criteria are in the teacher's domain. In view of this situation, it remains to say that in this project all the considerations required by the teacher are completely fulfilled.

## Steps

1. Access the AWS Kubernetes service  
   <img width="680" alt="MTsNW6qoDqvcjKaxznHQiVojfL6P1R6N-_Qp3SrNK9Yr7AsN10TQubZ5YKjo-ZmOFLofIadeLeRwpw6OktFB5exWrLL7bMbN6thURA6XxqpTR-raiirE6FwGyu9t" src="https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/52affb25-fd8d-4e05-b793-252e89ecdfb3">


2. Change our region  
   <img width="315" alt="US East (N  Virginia)" src="https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/3c82d5ad-3d2a-4942-ade4-2a6af52e873a">


3. Create our Kubernetes cluster  
   ![cL-THyQR_VY0hxT6j9FX-D0Pp16mmk0KC8VFt19gcLUgUDzzN8hDBTgyI6vf_tWLyjo8VxTnJhpb-t1yY5SzgwN5Ob8r9cWMr2VrlVodSKU7-a2ooFdC6341eQFN](https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/cc7d18ec-410b-4a30-a71f-649ed6cd46f1)


4. Configure the cluster in the Create Cluster section  
   ![Configure logging](https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/57b2dea3-5edb-4c0f-84b8-c2deb08f9f32)


5. Open the AWS Shell  
   ![vzSnhvYiqfWtkEt0-Nww3cV51ZIrgdWywk09R42vUpNvr7gn5et8gbFocfsl0tMsvCN4HeUgN0RiY2H4Aw9q3CjbCkCOQ42sm_LWKQg3RqZUGYBu5KSMIofF5K6D](https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/d9f96788-356e-4703-a2df-a2d38ef1e641)


6. Install Helm:  
   ```bash
   sudo yum install openssl -y
   curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
   chmod 700 get_helm.sh
   ./get_helm.sh
   ```

7. Install Bitnami:
   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```

8. Navigate to `.kube/config`. If the directory does not exist, create it with:  
   ```bash
   mkdir -p $HOME/.kube
   ```
   
   Then:
   ```bash
   nano config
   ```

9. Add the following to our `config` file (important to take the attributes from `my_cluster >> overview`):  
   <img width="553" alt="Details" src="https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/b015834a-4738-4a5f-91a6-d9e884743847">


   Content of the `config` file:
   ```yaml
   apiVersion: v1
   kind: Config
   clusters:
   - cluster:
       server: <API_URL>
       certificate-authority-data: <CERTIFICATE_AUTHORITY>
     name: eks_CLUSTER_NAME
   contexts:
   - context:
       cluster: eks_CLUSTER_NAME
       user: eks_CLUSTER_NAME
     name: eks_CLUSTER_NAME
   current-context: eks_CLUSTER_NAME
   users:
   - name: eks_CLUSTER_NAME
     user:
       exec:
         apiVersion: client.authentication.k8s.io/v1beta1
         command: aws
         args:
           - "eks"
           - "get-token"
           - "--cluster-name"
           - "<CLUSTER_NAME>"
         # - "--role"
         # - "ROLE_ARN" # Uncomment this line and replace with the ARN of the role to assume if you're using a role
         # env: # This is only needed if you need to set specific environment variables
         # - name: AWS_PROFILE
         #   value: "aws-profile" # Specify your AWS profile here if you're using a named profile
   ```

10. Add the following in `~/.bashrc`:  
    ```bash
    export KUBECONFIG=$HOME/.kube/config
    ```

11. Execute:  
    ```bash
    source ~/.bashrc
    ```

12. Now go to `my_cluster >> add ons` and add the following add-on  
    <img width="493" alt="Amazon EBS CSI Driver" src="https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/24878e3b-006e-47f4-b04e-2dfa3b923fc3">


13. Navigate to `my_cluster >> compute` and create a Node Group

14. Once it is finished creating, it will automatically bring up two nodes, which we can verify with the command:
    ```bash
    kubectl get nodes
    ```

15. Now we must install WordPress on our Kubernetes:
    1. For an internal service:
       ```bash
       helm install my-release --set wordpressUsername=admin --set wordpressPassword=password bitnami/wordpress
       ```
    2. For an external service with RDS:
       1. First, go to RDS and create our service  
          ![Create database](https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/10d0ede1-3420-44f5-8bec-3ad15516dd1d)


       2. Then execute in the console:
          ```bash
          helm install my-release bitnami/wordpress \
            --set wordpressUsername=admin \
            --set wordpressPassword=password \
            --set externalDatabase.host=your-rds-endpoint \
            --set externalDatabase.user=your-rds-username \
            --set externalDatabase.password=your-rds-password \
            --set externalDatabase.database=your-rds-database-name \
            --set mariadb.enabled=false
          ```

16. Check that our pods are running:  
    ```bash
    kubectl get pods
    ```

17. Check that our volumes are up:  
    ```bash
    kubectl get pvc
    ```

18. After the pods are running, go to Load Balancers, copy the DNS name and paste it into the browser  
  ![C3iJIsr3FJ-34Jt29MpnG0VeuPSQT7TtSndhB35HBaiAoO2b9FIRRE-fQ2OqCJmSVsjfw-a9kyg2MTlQS5jrh210ommX4FFLvqws2wE4KiIsNAovHdgbedBg7IDW](https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/dc695ed8-da9d-4828-b1fa-f05e8b410abf)


19. And with that, we will have our WordPress running  

    ![Mi primer kubernete](https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/ce038c07-305d-4d8e-bd92-c7808dda2a06)

# How to Set Up SSL Certificate with ACM
1. Once we have our DNS Name, we will use a domain name registration service to register the DNS of our Load Balancer. Example: NameCheap.
      ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/15c96133-69df-45a8-b5ba-2b75cb13a97c)
2. After securing our DNS (proyecto2.example), we go to AWS's `Certificate Manager` service:
      ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/a51bd90d-7334-49dd-9b43-ff96f5940be4)
3. We will click on the `Request` option to ask for a certificate for our DNS:
      ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/e27159a8-8615-4141-91ff-d839e3f9d62d)
4. We opt to request a public certificate and then click on next:
      ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/df459dce-4859-40d1-8e7d-0df58676c927)
5. We enter our assigned DNS in the configuration and finally click on request:
      ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/fe41cd9a-c6cb-4760-9678-950f0ef15b8d)
6. Next, we will go back to our domain name registration service and register our `CName name` and `CName Value`. This information can be found in our certificate requested in ACM:
      ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/4fc37740-b8a1-4f44-be60-4b6179ad7657)
7. Once our certificate is successfully created, we have finished this first part.
      ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/49a1e99d-397b-4dfd-ab01-38aeabb19e54)
8. The next step is to return to `EC2 >> Load Balancer` and open our Load Balancer:
      ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/016542c4-c622-4f7c-9dfb-04edf2325675)
9. Then we will go to Listeners >> Manage Listeners:
       ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/c7900ad1-2509-4f90-86f8-9f45dd912697)
10. Next, we will remove the TCP Port 443 configuration and create a new one for the HTTPS protocol. It's important that the `Instance port` of this new rule matches the one defined for TCP Port 80.
       ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/9a8423f6-853c-45ad-b083-3a6957ba1526)
11. In our new rule, under the `Default SSL/TLS Certificate` attribute, we will assign the certificate we created earlier:
       ![image](https://github.com/jdprietom03/topicos-tel-p2/assets/80794157/edfbaf25-03ac-4e4b-95ce-ccebf12602c2)
12. Finally, we save changes, go to our browser, and access using our created DNS but via https. Example: https://my-domain.example
       ![My first Kubernetes](https://github.com/Jguerra47/jsguerrah-st0263/assets/61121948/ce038c07-305d-4d8e-bd92-c7808dda2a06)









   


