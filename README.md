# **🚀 Deploying a Simple Web Application on Kubernetes (AWS EKS)**

This guide provides a step-by-step walkthrough of deploying an **Nginx** web application on **Amazon Elastic Kubernetes Service (EKS)** using the AWS CLI, kubectl, and eksctl. It includes setup, cluster creation, deployment, service exposure, and cleanup.

## **🧭 Flow Overview**

The process includes:

* Setting up AWS and IAM credentials.  
* Installing required CLI tools (aws, kubectl, eksctl).  
* Creating and configuring Kubernetes manifest files.  
* Launching an EKS cluster.  
* Deploying and exposing an Nginx application.  
* Accessing the application via browser.  
* Cleaning up AWS resources.

## **🧩 Prerequisites**

Before starting, ensure the following tools and accounts are set up on your system:

| Requirement | Description | Reference |
| :---- | :---- | :---- |
| **AWS Account** | Required to use AWS EKS and related services. | AWS Console |
| **AWS CLI** | Command-line tool to interact with AWS services. | AWS CLI Installation Guide |
| **IAM User** | User with policies to manage EKS resources. | See Step 1 |
| **kubectl** | Command-line tool for managing Kubernetes clusters. | Kubernetes CLI Setup |
| **eksctl** | Simplifies EKS cluster creation. | eksctl Setup |

## **🧑‍💻 Step 1: Configure AWS IAM User**

### **1.1 Create IAM User**

1. Log in to **AWS Management Console** → Navigate to **IAM**.  
2. Select **Users** → **Create user**.  
3. Enter username: Kubernetes-User-Admin.  
4. Click **Next**.

### **1.2 Attach Permissions**

Attach these managed policies:

* AdministratorAccess  
* AmazonEKSClusterPolicy  
* AmazonEKSServicePolicy  
* AmazonEC2ContainerRegistryReadOnly

### **1.3 Create Access Keys**

1. Click on your user → **Security Credentials** tab.  
2. Under Access Keys, click **Create Access Key**.  
3. Choose **Command Line Interface (CLI)**.  
4. Confirm and create.  
5. **Save your:**  
   * Access Key ID  
   * Secret Access Key

## **⚙️ Step 2: Configure AWS CLI**

In PowerShell:  
aws configure

Enter your credentials:  
AWS Access Key ID \[None\]: \<your-access-key\>  
AWS Secret Access Key \[None\]: \<your-secret-key\>  
Default region name \[None\]: us-east-1  
Default output format \[None\]: json

Verify configuration:  
aws \--version  
aws sts get-caller-identity

## **🧱 Step 3: Prepare Configuration Files**

Create a folder named kub-config-files on your Desktop. Inside it, create three files:

### **1️⃣ cluster-config.yaml**

apiVersion: eksctl.io/v1alpha5  
kind: ClusterConfig  
metadata:  
  name: my-cluster  
  region: us-east-1  
  version: "1.32"

managedNodeGroups:  
  \- name: my-nodes  
    instanceType: t3.small  
    desiredCapacity: 2  
    volumeSize: 20  
    ssh:  
      allow: false

### **2️⃣ nginx-deployment.yaml**

apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: nginx-deployment  
  labels:  
    app: nginx  
spec:  
  replicas: 3  
  selector:  
    matchLabels:  
      app: nginx  
  template:  
    metadata:  
      labels:  
        app: nginx  
    spec:  
      containers:  
      \- name: nginx-container  
        image: nginx:latest  
        ports:  
        \- containerPort: 80  
        resources:  
          limits:  
            memory: "128Mi"  
            cpu: "500m"

### **3️⃣ nginx-service.yaml**

apiVersion: v1  
kind: Service  
metadata:  
  name: nginx-service  
  labels:  
    app: nginx-service  
spec:  
  selector:  
    app: nginx  
  type: LoadBalancer  
  ports:  
    \- port: 80  
      targetPort: 80

## **🖥️ Step 4: Open Terminal in Configuration Folder**

Navigate to the kub-config-files folder in your terminal.  
Ensure you are in the correct directory by checking:  
dir

*(You should see the three YAML files.)*

## **🧩 Step 5: Install and Verify Tools**

1. **Verify AWS CLI**  
   aws \--version

2. **Check kubectl Installation**  
   kubectl version \--client

3. Install eksctl  
   If not installed, use Chocolatey:  
   choco install eksctl

   *Reboot and verify:*  
   eksctl version

## **🏗️ Step 6: Create Kubernetes Cluster on AWS EKS**

Run the cluster creation command:  
eksctl create cluster \-f cluster-config.yaml

🕓 This process takes **10–15 minutes**.  
Verify Cluster:  
kubectl get nodes

✅ **Expected:** You should see nodes with STATUS \= Ready.

## **📦 Step 7: Deploy Nginx Application**

Apply the deployment:  
kubectl apply \-f nginx-deployment.yaml

Check deployment:  
kubectl get deployments  
kubectl get pods

✅ **Expected:**  
nginx-deployment   3/3   Running   1m

## **🌍 Step 8: Expose the Application**

Create a LoadBalancer service:  
kubectl apply \-f nginx-service.yaml

Verify the service and retrieve the external hostname:  
kubectl get svc

**Expected Output Example:**  
NAME           TYPE          CLUSTER-IP      EXTERNAL-IP                             PORT(S)       AGE  
nginx-service  LoadBalancer  10.0.171.239    ab68f9736265e41b7997.elb.amazonaws.com  80:30007/TCP  1m

## **🌐 Step 9: Access Your Web Application**

In your browser, open the full hostname from the EXTERNAL-IP column:  
http://\<EXTERNAL-IP\>

**Example:**  
\[http://ab68f9736265e41b799786bfb0ddfadb-343796880.us-east-1.elb.amazonaws.com\](http://ab68f9736265e41b799786bfb0ddfadb-343796880.us-east-1.elb.amazonaws.com)

✅ **Expected Result:** You should see the Nginx Welcome Page — “Welcome to nginx\!”

## **🧹 Step 10: Clean Up AWS Resources**

After testing, **delete resources immediately** to avoid charges.

### **Delete Kubernetes Objects:**

kubectl delete \-f nginx-service.yaml  
kubectl delete \-f nginx-deployment.yaml

### **Delete EKS Cluster (and its underlying AWS components):**

eksctl delete cluster \--name my-cluster \--region us-east-1

*(Optional) If the eksctl delete command fails due to dependencies, delete the CloudFormation stacks manually:*  
\# Delete Node Group Stack first (example name)  
aws cloudformation delete-stack \--stack-name eksctl-my-cluster-nodegroup-my-nodes \--region us-east-1

\# Wait for Node Group deletion to complete, then delete the main cluster stack  
aws cloudformation delete-stack \--stack-name eksctl-my-cluster-cluster \--region us-east-1

## **✅ Summary**

| Step | Description |
| :---- | :---- |
| **1** | Configured AWS IAM user with EKS permissions |
| **2** | Installed and verified AWS CLI, kubectl, and eksctl |
| **3** | Created Kubernetes manifests (Cluster, Deployment, Service) |
| **4** | Launched an EKS cluster and verified node readiness |
| **5** | Deployed and exposed Nginx using a LoadBalancer |
| **6** | Accessed application externally |
| **7** | Cleaned up resources |

## **💡 Best Practices**

* Use IAM roles for secure automation instead of static keys.  
* Always enable EKS cluster logging for observability.  
* Use CloudFront for HTTPS traffic and caching.  
* Set up auto-scaling for production workloads.  
* Regularly clean unused clusters and LoadBalancers to reduce cost.

## **📘 Author Information**

Prepared by: John Michael Oliba  
Passionate about: DevOps, Systems Engineering, Administration, Graphics Design, Accounts & Finance  
Date: October 17, 2025  
Version: 1.0
