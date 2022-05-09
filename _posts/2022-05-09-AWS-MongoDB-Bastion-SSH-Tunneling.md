---
title: Connecting to MongoDB deployed on AWS using Bastion and SSH Tunneling from Local
author:
  name: Robin Reni
  link: https://github.com/robinreni96
date: 2022-05-09 20:00:00 +0800
categories: [Cloud]
tags: [Python]
math: true
mermaid: true
image:
  src: /assets/img/posts/mongodb-aws-logos.png
  width: 800
  height: 500
---
There are two major ways to deploy MongoDB on AWS one way is to directly deploy using **MongoDB Atlas** and another way is using **Self Managed AWS**. This blog is focused on how to connect locally to MongoDB deployed by Self Managed method.

> To know more about MongoDB AWS : [https://docs.aws.amazon.com/quickstart/latest/mongodb/welcome.html](https://docs.aws.amazon.com/quickstart/latest/mongodb/welcome.html)

### Self Managed MongoDB AWS Architecture
![MongoDB_AWS](/assets/img/posts/mongodb-architecture-on-aws.png) source: MongoDB AWS

You may deployed the MongoDB in AWS using **CloudFormation, Terraform or manually**. After the deployment you will be verifying by below methods
  * Connect to Bastion
    ![Bastion](https://docs.aws.amazon.com/quickstart/latest/mongodb/images/node-connection.png)
  * From the bastion host, use SSH to log in to one of the primary instances created by the Quick Start template:
    ```shell
    ssh ec2-user@<Primary-MongoDB-Instance>
    ```
  * Execute the following commands from the terminal:
    ```shell
    mongo
    use admin
    db.auth("admin", "YourAdminPassword")
    rs.printReplicationInfo()
    rs.status()
    ```

What if you need to connect to your MongoDB locally from any system. We cannot connect directly to Mongo instance because it will not have any public IP and only internal VPC instance can connect to it. Here comes the **SSH Tunneling using Bastion concept** to solve the case.Let me work you through how to connect easily.

### Step 1: SG Port Rules
After deploying MongoDB two instance will be created **LinuxBastion and PrimaryReplicaNode**. Go to their Security Groups and add **"Custom TCP"** type , **"27017 - 27030"** port range & **"Custom 0.0.0.0/0"** source to allow all networks to connect.
![MongoDB_SG](/assets/img/posts/Mongo-SG.png)

### Step 2: Add Keypair to Local
> While creating your stack, you may given keypair for security purpose if you are not given please ignore this step
Add the private key .pem to your local ssh.

```shell
ssh-add -K private-key.pem
```
Output
```shell
Identity added: private-key.pem (private-key.pem)
```
To verify the key, Run
```shell
ssh-add –L
```
Output
```shell
ssh-rsa BSIROFNzaC1yc2EAAAADAQABAAAAgQDHEXAMPLErl25NOrbhgIGQzyO+TYyqbbYEueiEL
cXtOQHgEFpMAbKSOE8SSnlxMxiCXwTKd5/lVnmgcbDwBpe7ayQ6idzjHfvoxPsFrI3QSJVQgyN
cx0RylX9IjcvJOyw== private-key.pem
```

### Step 3: Create the SSH Tunnel
Now we create a tunnel to connect local to your mongoDB via Bastion instance. Bastion instance has NAT gateway to accept external ssh request and can forward the port request to mongo instance.
```shell
ssh -A -L <local-port>:<Mongo-Primary-Internal-IP>:<Mongo-Primary-Instance-Port> <Bastion-User>@<Bastion-Public-IP> -N -v
```
Example
```shell
ssh -A -L 24000:10.0.33.246:27017 ec2-user@53.143.117.118 -N -v
```
> **Important Note: You need to run the command everytime when you need to connect DB from local**

### Step 4: MongoDB URI & Connection
Now the URI looks like this
```shell
mongodb://username:password@127.0.0.1:24000/database?directConnection=true
```
> Note: Make sure you logged into your Mongoinstance using Bastion and created a user and DB to connect

You can use the URI in **terminal or compass or NoSQL booster or Studio 3T** etc to connect to your DB

If you have any feedbacks please do share in the comment section below


### References:
* [https://www.giladpeleg.com/blog/bastion-host-ssh-tunneling-mongo-replica-set/](https://www.giladpeleg.com/blog/bastion-host-ssh-tunneling-mongo-replica-set/)
* [https://docs.aws.amazon.com/quickstart/latest/mongodb/welcome.html](https://docs.aws.amazon.com/quickstart/latest/mongodb/welcome.html)
* [https://aws-quickstart.s3.amazonaws.com/quickstart-mongodb/doc/MongoDB_on_the_AWS_Cloud.pdf](https://aws-quickstart.s3.amazonaws.com/quickstart-mongodb/doc/MongoDB_on_the_AWS_Cloud.pdf)
* [https://docs.aws.amazon.com/documentdb/latest/developerguide/connect-from-outside-a-vpc.html](https://docs.aws.amazon.com/documentdb/latest/developerguide/connect-from-outside-a-vpc.html)




