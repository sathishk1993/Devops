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

================================================================================================================


