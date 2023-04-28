# node-todo-cicd

Project Architecture:

<img width="1068" alt="nodejs" src="https://user-images.githubusercontent.com/122585172/235133400-128a77c1-eef7-4ebc-bc56-a1ddc2136fbf.png">


This project  is about the deployment of the Node.js application on the Amazon EKS Kubernetes cluster. We will set up a pipeline with Jenkins.

Jenkins GitHub Webhook automates the build and deployment of applications when any commit is done to the source code.

Setup an AWS EC2 Instance
Login to an AWS account using a user with admin privileges and ensure your region is set to us-east-1 N. Virginia.

Move to the EC2 console. Click Launch Instance.

For name use Jenkins-EC2.

![image](https://user-images.githubusercontent.com/122585172/235124848-b07ece62-83e9-4e57-bf89-610e403315fe.png)


Select t2.medium because we will be installing Jenkins and t2.micro will not be sufficient enough to set up Jenkins.

![image](https://user-images.githubusercontent.com/122585172/235124943-2b9d3b05-278a-4354-b3c0-5e4921b85094.png)


Create and download the key pair(private key and public key)


![image](https://user-images.githubusercontent.com/122585172/235124987-e38a6626-694d-4a49-95cd-c4e4a03ec155.png)



Configure Security Group - This is an important step because here we need to add Custom TCP Port 8080, if you do not add this port then you will not be able to access Jenkins using the public IP address of the AWS EC2 instance.

![image](https://user-images.githubusercontent.com/122585172/235125065-43b3343e-767d-4071-936c-d53c4fe10bd9.png)




Click on launch Instance and once EC2 Instance started, connect to it with EC2 Instance Connect.


![image](https://user-images.githubusercontent.com/122585172/235125126-df9a9b55-ddf2-4e94-81d5-53a09d201c57.png)





Install JDK on AWS EC2 Instance
Before following the below steps, assume you have already launched the ubuntu-based ec2 medium instance.

Check if you have java already installed on your EC2 machine by running the following command -

sudo apt-get update
java --version


sudo apt-get update
java --version


If this command indicates that Java is not found, then it’s not installed and you can proceed with the next steps.
sudo apt install openjdk-11-jre-headless -y
java --version

You can install java by running the following command.


sudo apt install openjdk-11-jre-headless -y
java --version


Install and Setup Jenkins
Step 1: Install Jenkins

Follow the steps for installing Jenkins on the EC2 instance.

sudo apt-get update
sudo apt-get install jenkins
sudo systemctl status jenkins


Step 2: Setup Jenkins

Now go to AWS dashboard -> EC2 -> Instances(running)and click on Jenkins-EC2

Copy Public IPv4 address.

Alright now we know the public IP address of the EC2 machine, so now we can access Jenkins from the browser using the public IP address followed by port 8080.

![image](https://user-images.githubusercontent.com/122585172/235125861-2da85f92-6a29-48ef-b5de-011ba7607848.png)


If you are installing Jenkins for the first time then you need to supply the initialAdminPassword and you can obtain it from

![image](https://user-images.githubusercontent.com/122585172/235125909-5fea9beb-67b0-49c0-b312-40b3637c6c62.png)


After completing the installation of the suggested plugin you need to set the First Admin User for Jenkins.

And now your Jenkins is ready for use

Update visudo and assign administration privileges to jenkins user
Now we have installed the Jenkins on the EC2 instance. To interact with the Kubernetes cluster Jenkins will be executing the shell script with the Jenkins user, so the Jenkins user should have an administration(superuser) role assigned forehand.

Let's add jenkins user as an administrator and also ass NOPASSWD so that during the pipeline run it will not ask for root password.

Open the file /etc/sudoers in vi mode

sudo vi /etc/sudoers




sudo vi /etc/sudoers
Add the following line at the end of the file


jenkins ALL=(ALL) NOPASSWD: ALL
After adding the line save and quit the file.
sudo su - jenkins

Now we can use Jenkins as root user and for that run the following command -



sudo su - jenkins
Install Docker
The docker installation will be done by the Jenkins user because now it has root user privileges.

Add jenkins user to Docker group. Jenkins will be accessing the Docker for building the application Docker images, so we need to add the Jenkins user to the docker group.

sudo apt install docker.io
docker --version
docker ps
sudo usermod -aG docker jenkins
sudo reboot
![image](https://user-images.githubusercontent.com/122585172/235127223-98f07bd8-2818-4086-beec-6d3ade12198a.png)
![image](https://user-images.githubusercontent.com/122585172/235127292-23bf4bf5-1974-45fd-a630-73864ac1ea41.png)




Install and Setup AWS CLI


Now we need to set up the AWS CLI on the EC2 machine so that we can use eksctl in the later stages

Let us get the installation done for AWS CLI 2
sudo apt install awscli
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update
aws --version



Okay now after installing the AWS CLI, let's configure the AWS CLI so that it can authenticate and communicate with the AWS environment.



aws configure
To configure the AWS the first command we are going to run is

Once you execute the above command it will ask for the following information -


Default output format [None]:

You can click on the Create New Access Key and it will let you generate - AWS Access Key ID, AWS Secret Access Key.

(Note: - Always remember you can only download your access id and secret once, if you misplace the secret and access then you need to recreate the keys again.



Install and Setup Kubectl
Moving forward now we need to set up the kubectl also onto the EC2 instance where we set up the Jenkins in the previous steps.

Here is the command for installing kubectl

curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version


Verify the kubectl installation by running the command kubectl version and you should see the following output.



Install and Setup eksctl
Okay, the first command which we are gonna run to install the eksctl

Download and extract the latest release of eksctl with the following command.

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
Move the extracted binary to /usr/local/bin.

sudo mv /tmp/eksctl /usr/local/bin
Test that your installation was successful with the following command.


eksctl version


Creating an Amazon EKS cluster using eksctl
Now in this step, we are going to create Amazon EKS cluster using eksctl

You need the following in order to run the eksctl command

Name of the cluster : --name first-eks-cluster1

Version of Kubernetes : --version 1.24

Region : --region us-east-1

Nodegroup name/worker nodes : --nodegroup-name worker-nodes

Node Type : --nodegroup-type t2.micro

Number of nodes: --nodes 2

Here is the eksctl command -


eksctl create cluster --name first-eks-cluster1 --version 1.24 --region us-east-1 --nodegroup-name worker-nodes --node-type t2.micro --nodes 2
It took me 20 minutes to complete this EKS cluster. If you get any error for not having sufficient data for mentioned availability zone then try it again.

Verify the EKS kubernetes cluster on AWS Console.

You can go back to your AWS dashboard and look for Elastic Kubernetes Service -> Clusters


![image](https://user-images.githubusercontent.com/122585172/235128129-a04a6c66-e4e5-477b-bc16-466f729d6191.png)

![image](https://user-images.githubusercontent.com/122585172/235128174-e73b0462-8737-4441-b356-5f1553abef89.png)



Add Docker and GitHub Credentials on Jenkins
Step 1: Setup Docker Hub Secret Text in Jenkins

You can set the docker credentials by going into -

Goto -> Jenkins -> Manage Jenkins -> Manage Credentials -> Stored scoped to jenkins -> global -> Add Credentials

![image](https://user-images.githubusercontent.com/122585172/235128251-217501fc-f344-4ccb-85ee-940a907ade9c.png)


Step 2. Setup GitHub Username and password into Jenkins

Now we add one more username and password for GitHub.

Goto -> Jenkins -> Manage Jenkins -> Manage Credentials -> Stored scoped to jenkins -> global -> Add Credentials

![image](https://user-images.githubusercontent.com/122585172/235128322-93001c22-990a-47cc-83eb-69f3c3378018.png)




Build, deploy and test CI/CD pipeline
Okay, now we can start writing out the Jenkins pipeline for deploying the Node.js Application into the Kubernetes Cluster.

Create new Pipeline: Goto Jenkins Dashboard or Jenkins home page click on New Item

![image](https://user-images.githubusercontent.com/122585172/235128361-a92a6b71-84ab-4f21-a4a4-40f441ff92ca.png)


Pipeline Name: Now enter Jenkins pipeline name and select Pipeline

![image](https://user-images.githubusercontent.com/122585172/235128410-1acabba4-91aa-475b-9fbe-196603822670.png)




Add pipeline script: Goto -> Configure and then pipeline section.



Copy the below script and paste into Pipeline Script.


node {

    stage("Git Clone"){

        git credentialsId: 'GIT_HUB_CREDENTIALS', url: 'your github url', branch: 'master' 
    }

     stage("Build") {

       sh 'docker build . -t shriniwas/node-todo-test:latest'
       sh 'docker image list'

    }

    withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'PASSWORD')]) {
        sh 'docker login -u shriniwas-p $PASSWORD'
    }

    stage("Push Image to Docker Hub"){
        sh 'docker push shriniwas/node-todo-test:latest'
    }

    stage("kubernetes deployment"){
        sh 'kubectl apply -f deployment.yml'
    }
}
To set up Jenkins - GitHub Webhook
Now, go to the “Build Triggers” tab.

Here, choose the “GitHub hook trigger for GITScm pulling” option, which will listen for triggers from the given GitHub repository, as shown in the image below.

![image](https://user-images.githubusercontent.com/122585172/235128635-84bf44e8-218e-4a1a-ab43-11eace48135f.png)


Jenkins GitHub Webhook is used to trigger the action whenever Developers commit something into the repository. It can automatically build and deploy applications.

Switch to your GitHub account, go to “Settings” option. Here, select the “Webhooks” option and then click on the “Add Webhook”

It will provide you the blank fields to add the Payload URL where you will paste your Jenkins address, Content type, and other configuration.

Go to your Jenkins tab and copy the URL then paste it in the text field named “Payload URL“, as shown in the image below. Append the “/github-webhook/” at the end of the URL.

![image](https://user-images.githubusercontent.com/122585172/235128718-4d195214-570b-4812-b8ba-eaf73496d744.png)






You completed Jenkins GitHub Webhook. Now for any commit in the GitHub repository, Jenkins will trigger the event specified




![image](https://user-images.githubusercontent.com/122585172/235128821-4c1ca941-591a-457b-b65f-ea83645852cb.png)



After pushing code to Github repository

![image](https://user-images.githubusercontent.com/122585172/235128941-01f553b7-6fd6-46c3-a1de-ef9bbe6c0902.png)

![image](https://user-images.githubusercontent.com/122585172/235129072-be955522-2f0b-48af-895e-9b82f5ea9f9d.png)


You can access the rest endpoint from a browser using the EXTERNAL-IP address.


![image](https://user-images.githubusercontent.com/122585172/235129203-6a19d3a6-b4b1-465f-8669-860545a0b844.png)

Clean up
Copy Deployment.yml file (From Github Repository) to EC2 server and run with below command.



kubectl delete -f deployment.yml
Delete EKS Cluster from AWS Console.

![image](https://user-images.githubusercontent.com/122585172/235129374-ead1e37a-7b36-4d36-a95e-428230086b4c.png)


Terminate EC2 Instance.

![image](https://user-images.githubusercontent.com/122585172/235129400-45ba9229-2fe4-47dc-970b-6e06e32bbec4.png)


Conclusion
We have successfully deployed our Node.js App in Amazon EKS cluster using AWS EC2, Jenkins, Docker, Kubernetes, GitHub, Webhook.

If you have liked this project and my effort please share this and fork my Github repo for more practice.
