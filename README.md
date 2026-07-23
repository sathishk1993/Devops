IN Jenkins Master - EC2 Instance
1. Install Java,Jenkins (Supported version openjdk21)

==============================================================================================

In Jenkins Agent - EC2 Instance (Refer the files and install)
1. Git with Jenkins
2. Install Java
3. Kubectl Installation
4. Trviy Integration with Jenkins
5. AWS CLI Installation
6. AWS Credentials Plugin
7. Docker with Jenkins
8. Install Maven -If applicable to the project level
9. Creation of user
10. Managed Kubernetes with Jenkins

======================================= THIS PART ABOUT JENKINS MASTER/CONTROLLER===================================================
To establish a connection from Jenkins Master to Agent Node

1. Open the Jenkins master EC2 instance
2. Copy the public key of the Jenkins master to the Agent EC2 instance and paste in authorized_keys
3. Copy the private key of Jenkins master and store it in Jenkins master (Go to Manage Jenkins->  Credentials-> SSH Username with private key
   Scope -> System(Jenkins and nodes only).
   ID -> some random meaningful name.
   Username-> some meaning name
  Click on private key and past the private key of the Jenkins master.

========================================= THIS PART ABOUT AGENT NODE CREATION AND SETUP================================================

1. Go to Manage Jenkins
2. Click on Nodes
3. Click on New nodes
4. Enter Node name
5. Type -> Permanent
6. Enter Name, Description, Number of executors and for especially to find the remote root directory use this command in  echo $HOME in agent node, you will get /home/<username>.
7. Give some Label name
8. Launch method -> Launch agents via ssh -> copy the private IP of the agent instance.
9. Choose the credentials which we stored in credentials (see the step3 in Master)
10. Host Key verification>for testing you can use Non verifying verification strategy.
11. Verify SSH connectivity from Jenkins Controller to Agent:

Command:
ssh <username>@<Agent-Private-IP>

Example:
ssh agent@10.0.0.5

Successful login without a password confirms the Controller can communicate with the Agent.

=============================================== MANAGED & INLINE POLICY  =================================================================

Managed Policy

Managed Policy = A standalone IAM policy that is created separately under IAM → Policies and then attached to one or more Users, Groups, or Roles.

Flow:

IAM
 │
 ├── Policies
 │      │
 │      ├── Create Policy
 │      └── Attach to
 │             ├── User
 │             ├── Group
 │             └── Role

Examples:
• Create a policy with S3 Read access.
• Attach it to a User.
• Attach the same policy to a Group.
• Attach the same policy to an IAM Role.

Golden Rule:
Managed Policy = Create Once → Reuse Many Times.
Inline Policy = Create Inside One User/Group/Role → Cannot Be Reused.


Inline Policy

1. Go to AWS Console.
2. Open IAM.
3. Click Roles.
4. Select your role (Example: JenkinsEC2Role).
5. Open the Permissions tab.
6. Click Add permissions.
7. Click Create inline policy.
8. Select the JSON tab.
9. Paste the policy JSON.
10. Click Next.
11. Enter the policy name (Example: JenkinsS3InlinePolicy).
12. Click Create policy.

Result:

* The policy is stored inside JenkinsEC2Role.
* It cannot be attached to any other role.

Example Policy JSON (used for both Managed and Inline)

{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": [
"s3",
"s3"
],
"Resource": "arn:aws:s3:::my-bucket/*"
}
]
}

Golden Rule

Managed Policy:
IAM → Policies → Create Policy → Attach to Role

Inline Policy:
IAM → Roles → Select Role → Add Permissions → Create Inline Policy

Managed Policy: A reusable IAM policy created under IAM → Policies that can be attached to multiple users, groups, or roles.
Inline Policy: A policy created directly inside a specific IAM user, group, or role that cannot be shared with others.


============================================= IAM ROLE WITH TEMPORARY CREDENTIALS====================================================


Create an IAM Role with sts:AssumeRole (EC2 Example)

1. Open AWS Console.
2. Go to IAM.
3. Click Roles.
4. Click Create role.
5. Select AWS Service.
6. Choose EC2.
7. Click Next.

Note:
AWS automatically creates the Trust Policy with:

* Principal: ec2.amazonaws.com
* Action: sts:AssumeRole

8. Attach the required permissions policy (AWS Managed or Customer Managed), for example:

   * AmazonS3ReadOnlyAccess
   * AmazonEC2ReadOnlyAccess
   * Your own custom managed policy

9. Click Next.

10. Enter the Role Name (Example: JenkinsEC2Role).

11. Click Create role.

Result:

* Trust Policy → Defines who can assume the role (EC2).
* Permissions Policy → Defines what the role can do (S3, EC2, ECR, etc.).
* When an EC2 instance is launched with this role, AWS STS automatically assumes the role and provides temporary credentials.

Trust Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Interview One-Liner:

Create an IAM Role, choose the AWS service (for example, EC2), AWS automatically creates the `sts:AssumeRole` trust policy, then attach the required permissions policy to define what the role can access.


========================================================CROSS ACCOUNT POLICY======================================================

Cross Account IAM Role Creation

1. Login to the Target AWS Account (the account containing the resource).
2. Go to IAM → Roles.
3. Click Create Role.
4. Under Trusted Entity Type, select AWS Account.
5. Enter the Source AWS Account ID (the account that needs access).
6. Click Next.
7. Attach the required Managed Policy (AWS Managed or Customer Managed), such as AmazonS3ReadOnlyAccess or a custom policy.
8. Enter the Role Name (Example: CrossAccountS3Role).
9. Click Create Role.

AWS automatically creates the Trust Policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<Source-Account-ID>:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
Example:

Source Account (111111111111)
        │
        │ AssumeRole
        ▼
Target Account (222222222222)
        │
        ▼
CrossAccountS3Role
        │
        ├── Trust Policy → Allows Account 111111111111
        └── Managed Policy → AmazonS3ReadOnlyAccess

Interview Answer:
For cross-account access, create an IAM Role in the target account, choose "AWS Account" as the trusted entity, specify the source AWS Account ID, attach the required managed policy, and AWS STS issues temporary credentials using AssumeRole.

=====================================================AmazonEC2ContainerRegistryPowerUser===========================================

IAM Policy: AmazonEC2ContainerRegistryPowerUser

Purpose:
Attach this policy to the EC2 IAM Role used by the Jenkins Agent. It allows Jenkins to authenticate with Amazon ECR and push/pull Docker images.

Jenkins Pipeline Flow:
1. Git Clone
2. Maven Build
3. docker build
4. docker tag
5. aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.ap-south-1.amazonaws.com
6. docker push <account-id>.dkr.ecr.ap-south-1.amazonaws.com/myapp:latest
7. Deploy to EKS

Key Points:
- AmazonEC2ContainerRegistryPowerUser allows Jenkins to:
  • Authenticate with ECR
  • Push Docker images
  • Pull Docker images
  • Upload image layers
  • Download image layers

- The IAM Role provides AWS permissions.
- The command `aws ecr get-login-password` generates a temporary ECR authentication token using the IAM Role.
- `docker login --username AWS --password-stdin` authenticates the Docker Engine with ECR.
- `--username AWS` is a fixed value required by Amazon ECR. It is NOT your Docker Hub username or IAM username.

Interview Answer:
"We attach the AmazonEC2ContainerRegistryPowerUser policy to the Jenkins EC2 instance so that Jenkins can authenticate with Amazon ECR and push/pull Docker images. The IAM role provides AWS permissions, and the `aws ecr get-login-password | docker login` command authenticates the Docker client to the ECR registry before pushing or pulling images."


===========================================================================JENKINS AGENT IAM Policy: AmazonEKSClusterPolicy================================


IAM Policy: AmazonEKSClusterPolicy (commonly used for learning/lab environments; in production Jenkins often uses a custom least-privilege policy)

Purpose:
Allows Jenkins to connect to and manage an Amazon EKS cluster.

Jenkins Pipeline Flow:
1. Configure kubectl to connect to the EKS cluster
2. Verify cluster connectivity
3. Deploy Kubernetes manifests
4. Verify deployment

Commands:

# Configure kubeconfig
aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster

# Verify connection
kubectl get nodes

# Deploy application
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Verify deployment
kubectl get pods
kubectl get svc

Interview Answer:
"We attach an IAM role with the required EKS permissions to the Jenkins agent. During the pipeline, Jenkins uses 'aws eks update-kubeconfig' to configure kubectl for the target EKS cluster, then deploys the application using 'kubectl apply -f'. Finally, it verifies the deployment using 'kubectl get pods' and 'kubectl get svc'."

====================Jenkins → EKS Deployment Setup without RBAC========================================================================================


Prerequisites:
- Jenkins is installed.
- Jenkins Agent has Docker, AWS CLI, and kubectl installed.
- EKS cluster is running.
- ECR repository exists.
- Jenkins EC2/Agent has an IAM Role attached.

------------------------------------------------------------
Step 1: Attach IAM Policies to the Jenkins IAM Role
------------------------------------------------------------

Required AWS permissions:
- AmazonEC2ContainerRegistryPowerUser (or a custom ECR policy)
- AmazonEKSClusterPolicy (or a custom policy including eks:DescribeCluster)

Verify: aws sts get-caller-identity

------------------------------------------------------------
Step 2: Create an EKS Access Entry
------------------------------------------------------------

AWS Console

Amazon EKS
→ Clusters
→ <cluster-name>
→ Access
→ Create Access Entry

Principal:
Select the Jenkins IAM Role

Type:
Standard

------------------------------------------------------------
Step 3: Configure Kubernetes Authorization (In case if we have assigned AmazonEKSClusterAdminPolicy Policy
to Jenkins AGENT No need of RBAC,Role,Role+Binding, No kubernetes access)
------------------------------------------------------------

Option A (Learning/Lab)
Associate:
AmazonEKSClusterAdminPolicy

------------------------------------------------------------
Step 4: Connect Jenkins to EKS
------------------------------------------------------------

aws eks update-kubeconfig \
--region ap-south-1 \
--name my-eks-cluster

Verify:  kubectl get nodes

------------------------------------------------------------
Step 5: Configure Jenkins Pipeline
------------------------------------------------------------

Pipeline stages:

1. Checkout code from GitHub
2. Build Docker image
3. Push image to ECR
4. Update kubeconfig
5. Deploy manifests

Commands:

docker build -t myapp .

aws ecr get-login-password \
| docker login --username AWS --password-stdin <ECR_URL>

docker push <ECR_URL>/myapp:latest

aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

------------------------------------------------------------
Step 6: Verify Deployment
------------------------------------------------------------
kubectl get deployments
kubectl get pods
kubectl get svc
kubectl describe pod <pod-name>

------------------------------------------------------------
Production Flow
------------------------------------------------------------

GitHub -> Jenkins Pipeline -> Build Docker Image -> Push Image to Amazon ECR -> aws eks update-kubeconfig -> EKS Access Entry Authentication -> Kubernetes RBAC Authorization -> kubectl apply -f deployment.yaml -> Pods Created in Amazon EKS

Interview Answer:

1. Attach the required IAM policy to the Jenkins EC2 IAM Role.
2. Create an EKS Access Entry for the Jenkins IAM Role.
3. Associate an EKS access policy (or use Kubernetes RBAC for least privilege).
4. Run:
   aws eks update-kubeconfig --region ap-south-1 --name my-eks-cluster
5. Deploy the application using:
   kubectl apply -f deployment.yaml
6. Verify using:
   kubectl get pods
   kubectl get svc


Example of the flow:
GitHub -> Jenkins downloads YAML files -> Jenkins uses IAM Role -> arn:aws:iam::123456789012:role/JenkinsRole -> AWS authenticates the IAM Role
EKS passes this identity to Kubernetes -> Kubernetes checks RoleBinding -> RoleBinding finds the matching IAM Role ARN -> RoleBinding gives access to the Role

=============================================== JENKINS AGENT IAM, ROLE, RBAC Jenkins flow ====================================================

# Role: Jenkins Deployer
# Namespace: dev

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: jenkins-deployer
  namespace: dev

rules:
# Core API Resources
- apiGroups: [""]
  resources:
    - pods
    - pods/log
    - services
    - endpoints
    - configmaps
    - secrets
    - serviceaccounts
    - persistentvolumeclaims
  verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete

# Workload Resources
- apiGroups: ["apps"]
  resources:
    - deployments
    - replicasets
    - statefulsets
    - daemonsets
  verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete

# Batch Resources
- apiGroups: ["batch"]
  resources:
    - jobs
    - cronjobs
  verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete

# Networking Resources
- apiGroups: ["networking.k8s.io"]
  resources:
    - ingresses
    - networkpolicies
  verbs:
    - get
    - list
    - watch
    - create
    - update
    - patch
    - delete

# RoleBinding
# Namespace: dev
# ============================================================

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-deployer-binding
  namespace: dev

subjects:
- kind: Group
  name: jenkins-deployers
  apiGroup: rbac.authorization.k8s.io

roleRef:
  kind: Role
  name: jenkins-deployer
  apiGroup: rbac.authorization.k8s.io


Just to understand the concept:  
Role permissions decide what Jenkins can do
Remember:
Role = What actions are allowed
Example: create deployment, delete pod, update service
RoleBinding = Who gets those permissions
Example: Jenkins IAM Role, developer user, service account
IAM Role ARN is only referenced in RoleBinding
The Role never contains the IAM ARN.
There is no synchronization between AWS IAM and Kubernetes. During every request, EKS authenticates the IAM Role, and Kubernetes uses the RoleBinding to match that identity and apply the required permissions.

================================================JENKINS PIPLEINE PATTERNS==========================================================================================
Why is this architecture popular?
 
- Jenkins is easy to manage independently.
- If the Kubernetes cluster has issues, Jenkins is still available.
- Jenkins can deploy to multiple EKS clusters (Development, QA, UAT, Production).
- Upgrading the EKS cluster does not affect Jenkins.
- Jenkins runs outside the Kubernetes cluster, so no Kubernetes ServiceAccount is required.
 
Do all companies use this?
 
No. The most common deployment patterns are:
 
Pattern 1: Jenkins on EC2 (Very Common)
 
GitHub
   │
   ▼
Jenkins (EC2)
   │
   ▼
IAM Role
   │
   ▼
EKS Access Entry
   │
   ▼
Kubernetes RBAC
   │
   ▼
Amazon EKS
 
Uses:
- IAM Role
- EKS Access Entry
- Kubernetes RBAC
- No ServiceAccount for Jenkins
 
Pattern 2: Jenkins Inside Amazon EKS
 
GitHub
   │
   ▼
Jenkins Pod
   │
   ▼
ServiceAccount
   │
   ├── Kubernetes RBAC
   │
   └── IAM Role (IRSA) (if AWS access is required)
   │
   ▼
Amazon EKS
 
Uses:
- ServiceAccount
- Role/ClusterRole
- RoleBinding/ClusterRoleBinding
- IAM Role (IRSA) for AWS access
 
Pattern 3: GitHub Actions / GitLab CI / Azure DevOps (Modern CI/CD)
 
GitHub Actions / GitLab CI / Azure DevOps
                 │
                 ▼
IAM Role (OIDC)
                 │
                 ▼
Amazon EKS
 
Many companies are moving to this model because they no longer need to maintain Jenkins servers while still securely deploying applications to Amazon EKS.
