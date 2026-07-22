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
      "Effect": "Allow",
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

