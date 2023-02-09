# Creating an EC2 instance with an Apache Web Server Using AWS CLI

For this project, I will be showing you how to create an EC2 instance with an Apache server using the AWS command-line interface. The AWS CLI allows you to interact with AWS services using commands in your shell.


### Prerequisites
* You will need the latest version of AWS CLI, I will be using version 2. Here you can find instructions to install: Installing or updating the latest version of the AWS CLI — AWS Command Line Interface (amazon.com)

* An IAM user account with administrative privileges, and access keys to log into.

* A command-line terminal.

* Vim text editor installed.

### Step 1
To check that you installed and have the latest version of AWS CLI, type in your command line:

```
aws --version
```

Depending on what operating system you are using, I will be using Windows, the response should look like something like this:


![image](https://user-images.githubusercontent.com/115881685/217720278-f5731669-a668-4b18-bbc3-cdd3b08cf970.png)


Then we will configure AWS CLI to interact with AWS.

```
aws configure
```


You will then be prompted to put in your AWS Access Key ID and AWS Secret Access Key for your specific user account. Then input your default region of choice, I chose us-east-1. Set the default output format to JSON.


![image](https://user-images.githubusercontent.com/115881685/217720526-9a042ba3-163b-4c7f-9754-c533d3dfffa1.png)


### Step 2
Next we will need to create a Key Pair in order to launch an EC2 instance from our AWS CLI. To create a Key Pair enter the following command, replacing the bold text with your own.

```
aws ec2 create-key-pair --key-name * ApacheProject * --query 'KeyMaterial' --output text | out-file -encoding ascii -filepath **ApacheProject.pem**
```


To ensure the Key pair was indeed created you can enter:

```
aws ec2 describe-key-pairs --key-name ApacheProject
```


![image](https://user-images.githubusercontent.com/115881685/217721394-62d7d935-93e6-49e0-8a27-4c1b3d321010.png)



Another way to validate that the Key Pair exists is to navigate to the EC2 dashboard in your AWS console, under Key pairs. You will see you have a new Key pair with the name you inputted into the command line.


### Step 3

In this step we will create a security group. Security groups act as a firewall to control the inbound and outbound traffic to your instance. In order to create a security group, we will first need the VPC ID. The VPC is your virtual private cloud that the security group will be assigned to. To find the VPC ID simply type in the command the following.


```
aws ec2 describe-vpcs
```


![image](https://user-images.githubusercontent.com/115881685/217721553-d2a3a721-90c4-46e0-89cb-2af2a211da97.png)



Copy the VPC ID as shown and input in the command to create the security group.



```
aws ec2 create-security-group --group-name **project-sg** --description **"My security group"** --vpc-id **vpc-05139aa9bdbe91e09**
```


The output will show a Group ID, be sure to copy this because we will use it in the next step.


![image](https://user-images.githubusercontent.com/115881685/217722340-b23451a3-130c-4fe5-89e2-5637780e718b.png)



### Step 4



```
aws ec2 authorize-security-group-ingress --group-id sg-05abaeef593678b3c  --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-05abaeef593678b3c  --protocol tcp --port 80 --cidr 0.0.0.0/0
```



![image](https://user-images.githubusercontent.com/115881685/217722887-cb682876-e6a9-455f-a3b7-7e388f788be0.png)




### Step 5

Before we can launch our instance we first will need to choose an AMI. An AMi, Amazon Machine Image, is a template that holds the software configurations needed to launch an instance. We can find the AMI ID by moving to our AWS console > EC2 > Instance > Launch an Instance > AMI.




![image](https://user-images.githubusercontent.com/115881685/217722941-4110c2f4-d76b-4b1d-8136-aa99161dee68.png)




As you can see here I chose a free tier eligible AMI, Amazon Linux 2. Copy the AMI ID and save it for the next step. The instance type will be a T2.micro which is also free tier eligible.




### Step 6


For this project, I am installing Apache on my EC2 instance. To have Apache installed and running on instance launch we will need to create a script. I will use vim, to create a new vim script:




```
vim apache_script.sh
```


You will be put in the empty file in the vim editor. Here you will input this script:



```
#!/bin/bash
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
```



To exit the editor press “esc” key, and enter :wq to write and quit.



### Step 7

Now to bring it all together, we will finally launch our EC2 instance!


```
aws ec2 run-instances --image-id **ami-0aa7d40eeae50c9a9** --count 1 --instance-type **t2.micro** --key-name **ApacheProject** --security-group-ids **sg-05abaeef593678b3c** --user-data file://**apache_script.sh**
```



![image](https://user-images.githubusercontent.com/115881685/217723754-f00db4f2-4628-49a7-9014-c9c47525c920.png)




Again the bold script is where you input your own information.


Navigate back to the AWS console > EC2 > Instances. You should see that you have an instance and that it is running!




![image](https://user-images.githubusercontent.com/115881685/217723812-35bdd5aa-f6cf-4e04-a78a-a0fb4daf4e25.png)




Once the instance is done initializing you can copy the Public IP address and paste it in a browser.



![image](https://user-images.githubusercontent.com/115881685/217723932-ae254eca-1270-49c3-8748-ffabd2154b86.png)




This page shows that it worked! We successfully launched an EC2 instance with an Apache web server all from the AWS CLI.

Now to ensure we are not charged for the instance we will need to terminate it. This can be done through the AWS command line as well running this command with your instance ID.




```
aws ec2 terminate-instances --instance-ids i-018cf3be3f4ca0b34
```



![image](https://user-images.githubusercontent.com/115881685/217724181-60ddc363-b03b-43d7-be20-5a4717af3b0d.png)


Take a look in your AWS console and the instance should be terminated.



![image](https://user-images.githubusercontent.com/115881685/217724276-73752515-74a5-4268-bf75-345e29c75b4e.png)



Thanks for following along!








