# Orchestrating containers across multiple Virtual Servers with Kubernetes. Part 1

+ Benefits of Docker containers: Lightweight, easy to deploy, fast startup, efficient resource usage, ideal for microservices.

+ Challenges of container management: Large number of containers, single point of failure risk.

+ Popular container orchestration tools: Kubernetes (most widely adopted), Docker Swarm, others.

+ Challenges of running containers across multiple nodes:
Resilience: Restarting failed containers on other nodes.
Inter-node communication: Enabling communication between containers on different nodes.

+ Container orchestration solves these challenges by:
A



**K8s installation options**

To successfully implement "K8s From-Ground-Up", the following and even more will be done by you as a K8s administrator:

+ Install and configure master (also known as control plane) components and worker nodes (or just nodes).

+ Apply security settings across the entire cluster (i.e., encrypting the data in transit, and at rest)
In transit encryption means encrypting communications over the network using HTTPS At rest encryption means encrypting the data stored on a disk

+ Plan the capacity for the backend data store etcd

+ Configure network plugins for the containers to communicate

+ Manage periodical upgrade of the cluster

+ Configure observability and auditing

**Note:** Unless you have any business or compliance restrictions, ALWAYS consider to use managed versions of K8s – Platform as a Service offerings, such as Azure Kubernetes Service (AKS), Amazon Elastic Kubernetes Service (Amazon EKS), or Google Kubernetes Engine (GKE) as they usually have better default security settings, and the costs for maintaining the control plane are very low.

Let us begin building out Kubernetes cluster from the ground

## Tools to be used and expected result of the Project 20

+ VM: AWS EC2
+ OS: Ubuntu 20.04 lts+
+ Docker Engine / containerd
+ kubectl console utility
+ cfssl and cfssljson utilities
+ Kubernetes cluster

## You will create 3 EC2 Instances, and in the end, we will have the following parts of the cluster properly configured:

+ One Kubernetes Master
+ Two Kubernetes Worker Nodes
+ Configured SSL/TLS certificates for Kubernetes components to communicate securely
+ Configured Node Network
+ Configured Pod Network


#Introducing Project Architecture

![alt text](./images/1.png)

# STEP 0-INSTALL CLIENT TOOLS BEFORE BOOTSTRAPPING THE CLUSTER

+ Spin up an EC2 Instance and install some tools. This instance will be the client workstation:

First, you will need some client tools installed and configurations made on your client workstation. In this case, my client workstation will be my local machine (WSL UBUNTU 20.04)

The following tools needs to be configured on the client

+ awscli – is a unified tool to manage your AWS services. Configure the access and secret keys on the client.

+ kubectl – this command line utility will be your main control tool to manage your K8s cluster.

+ cfssl – an open source toolkit for everything TLS/SSL from Cloudflare

+ cfssljson – a program, which takes the JSON output from the cfssl and writes certificates, keys, CSRs, and bundles to disk.

![alt text](./images/5.png)

**Install and configure AWS CLI**

Configure AWS CLI to access all AWS services used, for this you need to have a user with programmatic access keys configured in AWS Identity and Access Management (IAM)


Generate access keys and store them in a safe place.

On your local workstation download and install the [latest version of AWS CLI](https://aws.amazon.com/cli/)

![alt text](./images/6.png)

[To configure your AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) - run your shell (or cmd if using Windows) and run:

```
$ aws configure --profile %your_username%
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

Test your AWS CLI by running:

`aws ec2 describe-vpcs
`

and check if you can see VPC details

![alt text](./images/2.png)

## Install kubectl

Kubernetes cluster has a Web API that can receive HTTP/HTTPS requests, but it is quite cumbersome to curl an API each and every time you need to send some command, so kubectl command tool was developed to ease a K8s administrator’s life.

With this tool you can easily interact with Kubernetes to deploy applications, inspect and manage cluster resources, view logs and perform many more administrative operations.

**Mac OS X**

Download the binary

```
curl -o kubectl https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/darwin/amd64/kubectl
```

Make it executable

`chmod +x kubectl
`

Move to the Bin directory

`sudo mv kubectl /usr/local/bin/
`

**Linux Or Windows using Gitbash or similar tool**

Download the binary

```
wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
```
![alt text](./images/3.png)

`chmod +x kubectl
`

Move to the Bin directory

`sudo mv kubectl /usr/local/bin/
`

**Verify that `kubectl` version 1.21.0 or higher is installed:**

`kubectl version --client
`

the output should be like 

```
Client Version: version.Info{Major:"1", Minor:"20+", GitVersion:"v1.20.4-dirty", GitCommit:"e87da0bd6e03ec3fea7933c4b5263d151aafd07c", GitTreeState:"dirty", BuildDate:"2021-03-15T10:03:32Z", GoVersion:"go1.16.2", Compiler:"gc", Platform:"darwin/amd64"}
```

![alt text](./images/4.png)

**Install CFSSL and CFSSLJSON**

`cfssl` is an open source tool by **Cloudflare used** to setup a **Public Key Infrastructure** [(PKI Infrastructure)](https://en.wikipedia.org/wiki/Public_key_infrastructure) for generating, signing and bundling TLS certificates. In previous projects you have experienced the use of **Letsencrypt** for the similar use case. Here, `cfssl` will be configured as a Certificate Authority which will issue the certificates required to spin up a Kubernetes cluster.
Download, install and verify successful installation of `cfssl` and `cfssljson`:

**Mac OS X**

```
curl -o cfssl https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssl
curl -o cfssljson https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/darwin/cfssljson
```

`chmod +x cfssl cfssljson
`

```
sudo mv cfssl cfssljson /usr/local/bin/
```

If you have issues using the binaries directly, you should consider using the package manager Homebrew and that might be a better option:

```
brew install cfssl
```

Verify that `cfssl` version `1.4.1` or higher is installed:

`Output:` cfssl version

```
Version: 1.4.1
Runtime: go1.12.12
```

cfssljson --version

```
Version: 1.4.1
Runtime: go1.12.12
```

**Linux Or Windows using Gitbash or similar tool**

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
```

`chmod +x cfssl_1.6.3_linux_amd64 cfssljson_1.6.3_linux_amd64`

`sudo mv cfssl_1.6.3_linux_amd64 /usr/local/bin/cfssl`

`sudo mv cfssljson_1.6.3_linux_amd64 /usr/local/bin/cfssljson`

## AWS Cloud resources for Kubernetes Cluster

As we already know, we need some machines to run the control plane and the worker nodes. In this section, you will provision EC2 Instances required to run your K8s cluster. You can use Terraform for this. But it is highly recommended to start out first with manual provisioning using `awscli` and have thorough knowledge about the whole setup. After that, you can destroy the entire project and start all over again using Terraform. This manual approach will solidify your skills and give you the opportunity to face more challenges.

**Step 1** - Configure Network Infrastructure

Virtual Private Cloud - VPC

1. Create a directory named k8s-cluster-from-ground-up


2. Create a VPC and store the ID as a variable:

`VPC_ID=$(aws ec2 create-vpc --cidr-block 172.31.0.0/16 --output text --query 'Vpc.VpcId')`

3. Tag the VPC so that it is named:

`aws ec2 create-tags --resources ${VPC_ID} --tags Key=Name,Value=manual-k8s-cluster`

![alt text](./images/7.png)
![alt text](./images/8.png)

**Domain Name System - DNS**

Enable DNS support for your VPC

`aws ec2 modify-vpc-attribute --vpc-id ${VPC_ID} --enable-dns-support '{"Value": true}'`

Enable DNS support for hostnames

`aws ec2 modify-vpc-attribute --vpc-id ${VPC_ID} --enable-dns-hostnames '{"Value": true}'`

AWS Region

Set the required region

`AWS_REGION=us-east-1`

![alt text](./images/9.png)

Dynamic Host Configuration Protocol – DHCP

Configure DHCP Options Set

The Dynamic Host Configuration Protocol (DHCP) is a network management protocol employed in Internet Protocol networks. Its primary function is to autonomously allocate IP addresses and establish various communication parameters for devices linked to the network through a client-server architecture.

Upon VPC creation, AWS generates a DHCP option set with default domain-name-servers (AmazonProvidedDNS) and domain-name (region-specific), enabling DNS resolution for instances within the VPC.

EC2 instances get default names like `ip-123-45-67-89.region.compute.internal`, but you can customize them for better organization. as demonstrated below

```
DHCP_OPTION_SET_ID=$(aws ec2 create-dhcp-options --dhcp-configuration "Key=domain-name,Values=$AWS_REGION.compute.internal" "Key=domain-name-servers,Values=AmazonProvidedDNS" --output text --query 'DhcpOptions.DhcpOptionsId')
```

Tag the DHCP Option set

`aws ec2 create-tags --resources ${DHCP_OPTION_SET_ID} --tags Key=Name,Value=manual-k8s-cluster`

Associate the DHCP Option set with the VPC

`Associate the DHCP Option set with the VPC`

`aws ec2 associate-dhcp-options --dhcp-options-id ${DHCP_OPTION_SET_ID} --vpc-id ${VPC_ID}`

![alt text](./images/10.png)

Subnet

Create and tag the Subnet

`SUBNET_ID=$(aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 172.31.0.0/24 --output text --query 'Subnet.SubnetId')`

`aws ec2 create-tags --resources ${SUBNET_ID} --tags Key=Name,Value=manual-k8s-cluster`

Internet Gateway – IGW

Create the Internet Gateway and attach it to the VPC

`INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --output text --query 'InternetGateway.InternetGatewayId')`

`aws ec2 create-tags --resources ${INTERNET_GATEWAY_ID} --tags Key=Name,Value=manual-k8s-cluster`

Attach to VPC

`aws ec2 attach-internet-gateway --internet-gateway-id ${INTERNET_GATEWAY_ID} --vpc-id ${VPC_ID}`

![alt text](./images/11.png)

Route tables

Create route tables, associate the route table to subnet, and create a route to allow external traffic to the Internet through the Internet Gateway

`ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id ${VPC_ID} --output text --query 'RouteTable.RouteTableId')`

`aws ec2 create-tags --resources ${ROUTE_TABLE_ID} --tags Key=Name,Value=manual-k8s-cluster`

`aws ec2 associate-route-table --route-table-id ${ROUTE_TABLE_ID} --subnet-id ${SUBNET_ID}`

![alt text](./images/12.png)

Create a route to allow external traffic to the Internet through the Internet Gateway

`aws ec2 create-route --route-table-id ${ROUTE_TABLE_ID} --destination-cidr-block 0.0.0.0/0 --gateway-id ${INTERNET_GATEWAY_ID}`

![alt text](./images/13.png)

Security Groups

Configure security groups

Create the security group and store its ID in a variable

```
SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name manual-k8s-cluster --description "Kubernetes cluster security group" --vpc-id ${VPC_ID} --output text --query 'GroupId')
```

Create the NAME tag for the security group

```
$ SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name manual-k8s-cluster --description "Kubernetes cluster security group" --vpc-id ${VPC_ID} --output text --query 'GroupId')
```

Create the NAME tag for the security group

`$ aws ec2 create-tags --resources ${SECURITY_GROUP_ID} --tags Key=Name,Value=manual-k8s-cluster`

Create Inbound traffic for all communication within the subnet to connect on ports used by the master node(s)

```
$ aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --ip-permissions IpProtocol=tcp,FromPort=2379,ToPort=2380,IpRanges='[{CidrIp=172.31.0.0/24}]
```

Create inbound traffic to allow connections to the Kubernetes API Server (port 6443) listening on port 6443

```$ aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol tcp --port 6443 --cidr 0.0.0.0/0
```

Create inbound SSH traffic from any source. In a production environment, restrict access exclusively to desired IPs or CIDR ranges for connection.

`$ aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol tcp --port 22 --cidr 0.0.0.0/0`

Create ICMP ingress for all types

```
$ aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol icmp --port -1 --cidr 0.0.0.0/0
```

![alt text](./images/14.png)

Network Load Balancer

Create a network Load balancer

```
$ LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer --name manual-k8s-cluster --subnets ${SUBNET_ID} --scheme internet-facing --type network --output text --query 'LoadBalancers[].LoadBalancerArn')
```

![alt text](./images/15.png)
![alt text](./images/16.png)

Tagret Group

Create a target group acknowledging its lack of defined criteria at this stage due to the absence of real targets. it will be "unhealthy".

```
$ TARGET_GROUP_ARN=$(aws elbv2 create-target-group --name manual-k8s-cluster --protocol TCP --port 6443 --vpc-id ${VPC_ID} --target-type ip --output text --query 'TargetGroups[].TargetGroupArn')
```

![alt text](./images/17.png)
![alt text](./images/18.png)

Register targets: Similar to above, you will provide the IP addresses for registration purposes. These IP addresses will serve as targets when the nodes become available.

`$ aws elbv2 register-targets --target-group-arn ${TARGET_GROUP_ARN} --targets Id=172.31.0.1{0,1,2}`

![alt text](./images/19.png)
![alt text](./images/20.png)

Create a listener to listen for requests and forward to the target nodes on TCP port 6443

```
$ aws elbv2 create-listener --load-balancer-arn ${LOAD_BALANCER_ARN} --protocol TCP --port 6443 --default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN} --output text --query 'Listeners[].ListenerArn'
```

![alt text](./images/21.png)
![alt text](./images/22.png)

K8s Public Address

Get the Kubernetes Public address

```
$ KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers --load-balancer-arns ${LOAD_BALANCER_ARN} --output text --query 'LoadBalancers[].DNSName')
```

![alt text](./images/23.png)

CREATE COMPUTE RESOURCES

Install jq tool

jq is a lightweight and flexible command-line tool for processing and manipulating JSON data. It is commonly used in Unix-like operating systems to filter, transform, and format JSON data from various sources, including files, APIs, and other data streams. jq provides a wide range of features for querying and manipulating JSON, making it a powerful tool for tasks like parsing JSON, extracting specific data, and creating new JSON structures.

`$ sudo apt update && sudo apt install jq -y`

![alt text](./images/24.png)

AMI

Get an image to create EC2 instances

```
$ IMAGE_ID=$(aws ec2 describe-images --owners 099720109477 --filters 'Name=root-device-type,Values=ebs' 'Name=architecture,Values=x86_64' 'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*' | jq -r '.Images|sort_by(.Name)[-1]|.ImageId')
```
![alt text](./images/25.png)

SSH key-pair

Create SSH Key-Pair

`$ mkdir -p ssh`

`$ aws ec2 create-key-pair --key-name manual-k8s-cluster --output text --query 'KeyMaterial' > ssh/manual-k8s-cluster.id_rsa`

`$ chmod 600 ssh/manual-k8s-cluster.id_rsa`

![alt text](./images/26.png)

EC2 Instances for Controle Plane (Master Nodes)

Create 3 Master nodes

```
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name manual-k8s-cluster \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.1${i} \
    --user-data "name=master-${i}" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')
  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=manual-k8s-cluster-master-${i}"
done
```

![alt text](./images/27.png)
![alt text](./images/28.png)

EC2 Instances for Worker Nodes

Create 3 worker nodes

```
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name manual-k8s-cluster \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.2${i} \
    --user-data "name=worker-${i}|pod-cidr=172.20.${i}.0/24" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')
  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=manual-k8s-cluster-worker-${i}"
done
```

![alt text](./images/29.png)
![alt text](./images/30.png)

## PREPARE THE SELF-SIGNED CERTIFICATE AUTHORITY AND GENERATE TLS CERTIFICATES

The following components running on the Master node will require TLS certificates.

+ kube-controller-manager
+ kube-scheduler
+ etcd
+ kube-apiserver

The following components running on the Worker nodes will require TLS certificates.

+ kubelet
+ kube-proxy

Therefore, you will provision a PKI Infrastructure using cfssl which will have a Certificate Authority. The CA will then generate certificates for all the individual components.

Self-Signed Root Certificate Authority (CA)

Here, We will provision a CA that will be used to sign additional TLS certificates.

Create a directory and cd into it

`$ mkdir ca-authority && cd ca-authority`

```
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "Hammed-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```

![alt text](./images/31.png)

The 3 important files here are:

+ ca.pem – The Root Certificate
+ ca-key.pem – The Private Key
+ ca.csr – The Certificate Signing Request

Generating TLS Certificates For Client and Server

We will need to provision Client/Server certificates for all the components. We must have encrypted communication within the cluster. Therefore, the server here are the master nodes running the api-server component. While the client is every other component that needs to communicate with the api-server.

Now we have a certificate for the Root CA, we can then begin to request more certificates which the different Kubernetes components, i.e. clients and server, will use to have encrypted communication.

Remember, the clients here refer to every other component that will communicate with the api-server. These are:

+ kube-controller-manager
+ kube-scheduler
+ etcd
+ kubelet
+ kube-proxy
+ Kubernetes Admin User

Let us begin with the Kubernetes API-Server Certificate and Private Key

The certificate for the Api-server must have IP addresses, DNS names, and a Load Balancer address included. Otherwise, we will have a lot of difficulties connecting to the api-server.

Generate the Certificate Signing Request (CSR), Private Key and the Certificate for the Kubernetes Master Nodes.

```
{
cat > master-kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
   "hosts": [
   "127.0.0.1",
   "172.31.0.10",
   "172.31.0.11",
   "172.31.0.12",
   "ip-172-31-0-10",
   "ip-172-31-0-11",
   "ip-172-31-0-12",
   "ip-172-31-0-10.${AWS_REGION}.compute.internal",
   "ip-172-31-0-11.${AWS_REGION}.compute.internal",
   "ip-172-31-0-12.${AWS_REGION}.compute.internal",
   "${KUBERNETES_PUBLIC_ADDRESS}",
   "kubernetes",
   "kubernetes.default",
   "kubernetes.default.svc",
   "kubernetes.default.svc.cluster",
   "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "Hammed-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  master-kubernetes-csr.json | cfssljson -bare master-kubernetes
}
```

![alt text](./images/32.png)

Creating the other certificates: for the following Kubernetes components:

+ Scheduler Client Certificate
+ Kube Proxy Client Certificate
+ Controller Manager Client Certificate
+ Kubelet Client Certificates
+ K8s admin user Client Certificate

kube-scheduler Client - Certificate and Private Key

```
{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:kube-scheduler",
      "OU": "Hammed-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```

kube-proxy Client - Certificate and Private Key

```
{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:node-proxier",
      "OU": "Hammed-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```

kube-controller-manager - Client Certificate and Private Key

```
{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:kube-controller-manager",
      "OU": "Hammed-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```

kubelet Client - Certificate and Private Key

Similar to how we configured the api-server's certificate, Kubernetes requires that the hostname of each worker node is included in the client certificate.

Also, Kubernetes uses a special-purpose authorization mode called Node Authorizer, that specifically authorizes API requests made by kubelet services. In order to be authorized by the Node Authorizer, kubelets must use a credential that identifies them as being in the system:nodes group, with a username of system:node:. Notice the "CN": "system:node:${instance_hostname}", in the below code.

Therefore, the certificate to be created must comply to these requirements. In the below example, there are 3 worker nodes, hence we will use bash to loop through a list of the worker nodes’ hostnames, and based on each index, the respective Certificate Signing Request (CSR), private key and client certificates will be generated.

```
for i in 0 1 2; do
  instance="manual-k8s-cluster-worker-${i}"
  instance_hostname="ip-172-31-0-2${i}"
  cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance_hostname}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:nodes",
      "OU": "Hammed-projects",
      "ST": "London"
    }
  ]
}
EOF

  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')

  internal_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PrivateIpAddress')

  cfssl gencert \
    -ca=ca.pem \
    -ca-key=ca-key.pem \
    -config=ca-config.json \
    -hostname=${instance_hostname},${external_ip},${internal_ip} \
    -profile=kubernetes \
    manual-k8s-cluster-worker-${i}-csr.json | cfssljson -bare manual-k8s-cluster-worker-${i}
done
```

kubernetes admin user - Client Certificate and Private Key

```
{
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:masters",
      "OU": "Hammed-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
}
```

Token Controller - certificate and private key

We need to generate certificate and private key for the Token Controller - a part of the Kubernetes Controller Manager.

kube-controller-manager responsible for generating and signing service account tokens which are used by pods or other resources to establish connectivity to the api-server.

```
{

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "Hammed-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
}
```

![alt text](./images/33.png)

### DISTRIBUTING THE CLIENT AND SERVER CERTIFICATES

Now it is time to start sending all the client and server certificates to their respective instances.

Let us begin with the worker nodes:

Copy these files securely to the worker nodes using scp utility

+ Root CA certificate – ca.pem
+ X509 Certificate for each worker node
+ Private Key of the certificate for each worker node

```
for i in 0 1 2; do
  instance="manual-k8s-cluster-worker-${i}"
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/manual-k8s-cluster.id_rsa \
    ca.pem ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done
```
![alt text](./images/34.png)

Master or Controller node: – Note that only the api-server related files will be sent over to the master nodes.

```
for i in 0 1 2; do
instance="manual-k8s-cluster-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/manual-k8s-cluster.id_rsa \
    ca.pem ca-key.pem service-account-key.pem service-account.pem \
    master-kubernetes.pem master-kubernetes-key.pem ubuntu@${external_ip}:~/;
done
```

![alt text](./images/35.png)

The kube-proxy, kube-controller-manager, kube-scheduler and kubelet client certificates will be used to generate client authentication configuration files later on.

## GENERATE KUBERNETES CONFIGURATION FILES FOR AUTHENTICATION USING 'KUBECTL'

In this step, We will create some files known as kubeconfig, which enables Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

You will need a client tool called kubectl to do this. And, by the way, most of your time with Kubernetes will be spent using kubectl commands.

Now it’s time to generate kubeconfig files for the kubelet, controller manager, kube-proxy, and scheduler clients and then the admin user.

First, let us create a few environment variables for reuse by multiple commands.

```
$ KUBERNETES_API_SERVER_ADDRESS=$(aws elbv2 describe-load-balancers --load-balancer-arns ${LOAD_BALANCER_ARN} --output text --query 'LoadBalancers[].DNSName')
```

Generate the kubelet kubeconfig file

For each of the nodes running the kubelet component, it is very important that the client certificate configured for that node is used to generate the kubeconfig. This is because each certificate has the node’s DNS name or IP Address configured at the time the certificate was generated. It will also ensure that the appropriate authorization is applied to that node through the Node Authorizer

Run the code below in the directory where all the certificates were generated.

```
for i in 0 1 2; do

instance="manual-k8s-cluster-worker-${i}"
instance_hostname="ip-172-31-0-2${i}"

 # Set the kubernetes cluster in the kubeconfig file
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://$KUBERNETES_API_SERVER_ADDRESS:6443 \
    --kubeconfig=${instance}.kubeconfig

# Set the cluster credentials in the kubeconfig file
  kubectl config set-credentials system:node:${instance_hostname} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

# Set the context in the kubeconfig file
  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=system:node:${instance_hostname} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

![alt text](./images/36.png)

List the output

`$ ls -ltr *.kubeconfig`

![alt text](./images/37.png)

Open up the kubeconfig files generated and review the 3 different sections that have been configured:

+ Cluster
+ Credentials
+ Kube Context

Kubeconfig file is used to organize information about clusters, users, namespaces and authentication mechanisms. By default, kubectl looks for a file named config in the $HOME/.kube directory. You can specify other kubeconfig files by setting the KUBECONFIG environment variable or by setting the --kubeconfig flag

Context part of kubeconfig file defines three main parameters: cluster, namespace and user. You can save several different contexts with any convenient names and switch between them when needed.

`$ kubectl config use-context %context-name%`

Generate the kube-proxy kubeconfig

```
{
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```

![alt text](./images/38.png)

Generate the Kube-Controller-Manager kubeconfig

Notice that the --server is set to use 127.0.0.1. This is because, this component runs on the API-Server so there is no point routing through the Load Balancer.

```
{
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```

![alt text](./images/39.png)

Generating the Kube-Scheduler Kubeconfig

```
{
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```

![alt text](./images/40.png)

Generate the kubeconfig file for the admin user

```
{
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

![alt text](./images/41.png)

Distribute the files to their respective servers using scp and a for loop.

For Worker nodes

```
for i in 0 1 2; do
  instance="manual-k8s-cluster-worker-${i}"
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/manual-k8s-cluster.id_rsa \
    kube-proxy.kubeconfig ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done
```

![alt text](./images/42.png)

For Master nodes

```
for i in 0 1 2; do
instance="manual-k8s-cluster-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/manual-k8s-cluster.id_rsa \
    ca.pem ca-key.pem service-account-key.pem service-account.pem \
    kube-controller-manager.kubeconfig kube-scheduler.kubeconfig admin.kubeconfig ubuntu@${external_ip}:~/;
done
```

![alt text](./images/43.png)

### PREPARE THE ETCD DATABASE FOR ENCRYPTION AT RES

Prepare the etcd database for encryption at rest.

Kubernetes uses etcd (A distributed key value store) to store variety of data which includes the cluster state, application configurations, and secrets. By default, the data that is being persisted to the disk is not encrypted. Any attacker that is able to gain access to this database can exploit the cluster since the data is stored in plain text. Hence, it is a security risk for Kubernetes that needs to be addressed.

To mitigate this risk, we must prepare to encrypt etcd at rest. "At rest" means data that is stored and persists on a disk. Anytime you hear "in-flight" or "in transit" refers to data that is being transferred over the network. "In-flight" encryption is done through TLS.

Generate the encryption key and encode it using "base64"

Kubernetes uses a 32-byte (256-bit) encryption key for etcd encryption. Etcd is a distributed key-value store used by Kubernetes to store configuration data and other distributed system information securely. The 32-byte key is typically used for encryption in etcd to ensure the confidentiality and integrity of the stored data. This key is generated and managed as part of the Kubernetes cluster's configuration for security purposes.

`$ ETCD_ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)`

`echo $ETCD_ENCRYPTION_KEY`

See the output below:

![alt text](./images/44.png)

```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ETCD_ENCRYPTION_KEY}
      - identity: {}
EOF
```
![alt text](./images/45.png)

Send the encryption file to the Controller nodes using scp and a for loop

```
for i in 0 1 2; do
instance="manual-k8s-cluster-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ssh/manual-k8s-cluster.id_rsa \
    encryption-config.yaml ubuntu@${external_ip}:~/;
done
```

![alt text](./images/46.png)

Bootstrap "etcd" cluster

The primary purpose of the etcd component is to store the state of the cluster. This is because Kubernetes itself is stateless. Therefore, all its stateful data will persist in etcd. Since Kubernetes is a distributed system – it needs a distributed storage to keep persistent data in it. etcd is a highly-available key value store that fits the purpose. All K8s cluster configurations are stored in a form of key value pairs in etcd, it also stores the actual and desired states of the cluster. etcd cluster is intelligent enough to watch for changes made on one instance and almost instantly replicate those changes to the rest of the instances, so all of them will always be  reconciled.

SSH into the controller server

Open three terminals in MobaXterm, SSH into the client-workstation and from each of these terminals, establish SSH connections to the different master nodes. Master 1

```
master_1_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=manual-k8s-cluster-master-0" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ssh/manual-k8s-cluster.id_rsa ubuntu@${master_1_ip}
```

```
master_2_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=manual-k8s-cluster-master-1" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ssh/manual-k8s-cluster.id_rsa ubuntu@${master_2_ip}
```

```
master_3_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=manual-k8s-cluster-master-2" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ssh/manual-k8s-cluster.id_rsa ubuntu@${master_3_ip}
```

Run the command

`$ ls -ltr`

You should be able to see all the files that have been sent to the nodes

Download and install etcd on each of the terminals using the multi-paste button.

```
wget https://github.com/etcd-io/etcd/releases/download/v3.4.15/etcd-v3.4.15-linux-amd64.tar.gz
```

Extract and install the etcd server and the etcdctl command line utility

```
{
tar -xvf etcd-v3.4.15-linux-amd64.tar.gz
sudo mv etcd-v3.4.15-linux-amd64/etcd* /usr/local/bin/
}
```

Configure the etcd server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem master-kubernetes-key.pem master-kubernetes.pem /etc/etcd/
}
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance

`$ export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)`

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to node Private IP address so it will uniquely identify the machine

```
$ ETCD_NAME=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)

echo ${ETCD_NAME}
```

Create the etcd.service systemd unit file

Read the documentation [here.](https://www.bookstack.cn/read/etcd-3.2.17-en/717bafd59fa87192.md)

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster master-0=https://172.31.0.10:2380,master-1=https://172.31.0.11:2380,master-2=https://172.31.0.12:2380 \\
  --cert-file=/etc/etcd/master-kubernetes.pem \\
  --key-file=/etc/etcd/master-kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/master-kubernetes.pem \\
  --peer-key-file=/etc/etcd/master-kubernetes-key.pem \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Start and enable the etcd Server

```
{
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
}
```

Verify the etcd installation

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/master-kubernetes.pem \
  --key=/etc/etcd/master-kubernetes-key.pem
  
```

Check the status

`$ sudo systemctl status etcd`

![alt text](./images/47.png)

### BOOTSTRAP THE CONTROL PLANE

Configure the components for the control plane on the master/controller nodes.

Create the Kubernetes configuration directory

`$ sudo mkdir -p /etc/kubernetes/config`

Download the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-apiserver" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-controller-manager" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-scheduler" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl"
```

Install the Kubernetes binaries

```
{
chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

Configure the Kubernetes API Server

```
{
sudo mkdir -p /var/lib/kubernetes/

sudo mv ca.pem ca-key.pem master-kubernetes-key.pem master-kubernetes.pem \
service-account-key.pem service-account.pem \
encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

`$ export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)`

Create the kube-apiserver.service systemd unit file Ensure to read each startup flag used in below systemd file from the documentation here

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/master-kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/master-kubernetes-key.pem\\
  --etcd-servers=https://172.31.0.10:2379,https://172.31.0.11:2379,https://172.31.0.12:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/master-kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --runtime-config='api/all=true' \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-account-signing-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-account-issuer=https://${INTERNAL_IP}:6443 \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/master-kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Configure the Kubernetes Controller Manager

Move the kube-controller-manager kubeconfig into place

`$ sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/`

Export some variables to retrieve the vpc_cidr – This will be required for the bind-address flag

```
export AWS_METADATA="http://169.254.169.254/latest/meta-data"
export EC2_MAC_ADDRESS=$(curl -s $AWS_METADATA/network/interfaces/macs/ | head -n1 | tr -d '/')
export VPC_CIDR=$(curl -s $AWS_METADATA/network/interfaces/macs/$EC2_MAC_ADDRESS/vpc-ipv4-cidr-block/)
export NAME=manual-k8s-cluster
```

Create the kube-controller-manager.service systemd unit file

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --cluster-cidr=${VPC_CIDR} \\
  --cluster-name=${NAME} \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authentication-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authorization-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Configure the Kubernetes Scheduler

Move the kube-scheduler kubeconfig into place

`$ sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/`

`$ sudo mkdir -p /etc/kubernetes/config`

Create the kube-scheduler.yaml configuration file

```
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```
Create the kube-scheduler.service systemd unit file

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Start the Controller Services

```
{
sudo systemctl daemon-reload
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

Check the status of the services. Start with the kube-scheduler and kube-controller-manager. It may take up to 20 seconds for kube-apiserver to be fully loaded.

```
{
sudo systemctl status kube-apiserver
sudo systemctl status kube-controller-manager
sudo systemctl status kube-scheduler
}
```

![alt text](./images/48.png)
![alt text](./images/49.png)
![alt text](./images/50.png)

Test that Everything is working fine.

To get the cluster details run

`$ kubectl cluster-info  --kubeconfig admin.kubeconfig`

![alt text](./images/51.png)

To get the current namespaces

`$ kubectl get namespaces --kubeconfig admin.kubeconfig`

![alt text](./images/52.png)

To reach the Kubernetes API Server publicly

`$ kubectl get componentstatuses --kubeconfig admin.kubeconfig`

![alt text](./images/53.png)
![alt text](./images/54.png)

On one of the controller nodes, configure Role Based Access Control (RBAC) so that the api-server has necessary authorization for for the kubelet.

Create the ClusterRole

```
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

Create the ClusterRoleBinding to bind the kubernetes user with the role created above

```
cat <<EOF | kubectl --kubeconfig admin.kubeconfig  apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```
![alt text](./images/55.png)

Configuring the Kubernetes Worker nodes

The Kubernetes API Server authenticates with the kubelet as the 'kubernetes' user, utilizing the 'kubernetes.pem' certificate for authentication.

To establish Role-Based Access Control (RBAC) for Kubelet Authorization, follow these steps:

+ Set up RBAC permissions to grant the Kubernetes API Server access to the Kubelet API on every worker node. This access is essential for tasks such as retrieving metrics, logs, and executing commands within pods.

+  Create the 'system:kube-apiserver-to-kubelet' ClusterRole, which includes permissions for accessing the Kubelet API and executing common operations related to pod management on the worker nodes.

To initiate this configuration, execute the provided script on the Controller node.

```
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

![alt text](./images/56.png)

Bind the system:kube-apiserver-to-kubelet ClusterRole to the kubernetes user so that API server can authenticate successfully to the kubelets on the worker nodes

```
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

Bootstraping components on the worker nodes

The following components will be installed on each node

+ kubelet
+ kube-proxy
+ Containerd or Docker
+ Networking plugins

SSH into the worker nodes

Worker-1

```
$ worker_1_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=manual-k8s-cluster-worker-0" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ssh/manual-k8s-cluster.id_rsa ubuntu@${worker_1_ip}
```
Worker-2

```
$ worker_2_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=manual-k8s-cluster-worker-1" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ssh/manual-k8s-cluster.id_rsa ubuntu@${worker_2_ip}
```

Worker-3

```
$ worker_3_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=manual-k8s-cluster-worker-2" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ssh/manual-k8s-cluster.id_rsa ubuntu@${worker_3_ip}
```

Install OS dependencies

```
{
  sudo apt-get update
  sudo apt-get -y install socat conntrack ipset
}
```

More about the dependencies:

+ Socat is the default implementation for Kubernetes port-forwarding when using dockershim for the kubelet runtime.

+ Dockershim was a temporary solution proposed by the Kubernetes community to add support for Docker so that it could serve as its container runtime. You should always remember that Kubernetes can use different container runtime to run containers inside its pods. For many years, Docker has been adopted widely and has been used as the container runtime for kubernetes. Hence the implementation that allowed docker is called the Dockershim. If you check the source code of Dockershim, you will see that socat was used to implement the port-forwarding functionality.

+ conntrack Connection tracking (“conntrack”) is a core feature of the Linux kernel’s networking stack. It allows the kernel to keep track of all logical network connections or flows, and thereby identify all of the packets which make up each flow so they can be handled consistently together. It is essential for performant complex networking of Kubernetes where nodes need to track connection information between thousands of pods and services.

+ ipset is an extension to iptables which is used to configure firewall rules on a Linux server. ipset is a module extension to iptables that allows firewall configuration on a "set" of IP addresses. Compared with how iptables does the configuration linearly, ipset is able to store sets of addresses and index the data structure, making lookups very efficient, even when dealing with large sets. Kubernetes uses ipsets to implement a distributed firewall solution that enforces network policies within the cluster. This can then help to further restrict communications across pods or namespaces. For example, if a namespace is configured with DefaultDeny isolation type (Meaning no connection is allowed to the namespace from another namespace), network policies can be configured in the namespace to whitelist the traffic to the pods in that namespace.

Kubernetes Network Policy And How It Is Implemented

Kubernetes network policies are application centric compared to infrastructure/network centric standard firewalls. There are no explicit CIDR or IP used for matching source or destination IP’s. Network policies build up on labels and selectors which are key concepts of Kubernetes that are used for proper organization (for e.g dedicating a namespace to data layer and controlling which app is able to connect there). A typical network policy that controls who can connect to the database namespace will look like below

```
apiVersion: extensions/v1beta1
kind: NetworkPolicy
metadata:
  name: database-network-policy
  namespace: tooling-db
spec:
  podSelector:
    matchLabels:
      app: mysql
  ingress:
   - from:
     - namespaceSelector:
       matchLabels:
         app: tooling
     - podSelector:
       matchLabels:
       role: frontend
   ports:
     - protocol: tcp
     port: 3306
```

***NOTE:*** Best practice is to use solutions like RDS for database implementation

Disable Swap

If swap is not disabled, kubelet will not start. It is highly recommended to allow Kubernetes to handle resource allocation.

Test if swap is already enabled on the host

`$ sudo swapon --show`

If there is no output, then you are good to go. Otherwise, run below command to turn it off

`$ sudo swapoff -a`

Download and install a container runtime. (Docker Or Containerd)

Containerd

Download binaries for runc, cri-ctl, and containerd

```
 wget https://github.com/opencontainers/runc/releases/download/v1.0.0-rc93/runc.amd64 \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.21.0/crictl-v1.21.0-linux-amd64.tar.gz \
  https://github.com/containerd/containerd/releases/download/v1.4.4/containerd-1.4.4-linux-amd64.tar.gz 
```

Configure containerd

```
{
  mkdir containerd
  tar -xvf crictl-v1.21.0-linux-amd64.tar.gz
  tar -xvf containerd-1.4.4-linux-amd64.tar.gz -C containerd
  sudo mv runc.amd64 runc
  chmod +x  crictl runc  
  sudo mv crictl runc /usr/local/bin/
  sudo mv containerd/bin/* /bin/
}
```

`$ sudo mkdir -p /etc/containerd/`

```
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
EOF
```

Create the containerd.service systemd unit file

```
cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

Create directories to configure kubelet, kube-proxy, cni, and a directory to keep the kubernetes root ca file

```
sudo mkdir -p \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

Download and Install Container Network Interface (CNI)

Container Network Interface (CNI), a Cloud Native Computing Foundation project, consists of a specification and libraries for writing plugins to configure network interfaces in Linux containers. It also comes with a number of plugins.

Kubernetes uses CNI as an interface between network providers and Kubernetes Pod networking. Network providers create network plugin that can be used to implement the Kubernetes networking, and includes additional set of rich features that Kubernetes does not provide out of the box.

Download the plugins available from [containernetworking’s](https://github.com/containernetworking/cni) GitHub repo and read more about CNIs and why it is being developed.

```
wget https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz
```

Install CNI into /opt/cni/bin/

`$ sudo tar -xvf cni-plugins-linux-amd64-v0.9.1.tgz -C /opt/cni/bin/`

There are few other plugins that are not included in the CNI, which are also widely used in the industry. They all have their unique implementation approach and set of features. such as

+ Calico
+ Weave Net
+ flanne etc

Sometimes you can combine more than one plugin together to maximize the use of features from different providers. Or simply use a CNI network provider such as canal that gives you the best of Flannel and Calico.

Download binaries for kubectl, kube-proxy, and kubelet

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubelet
```

Install the downloaded binaries

```
{
  chmod +x  kubectl kube-proxy kubelet  
  sudo mv  kubectl kube-proxy kubelet /usr/local/bin/
}
```

Configure the worker nodes components

Configure kubelet

In the home directory, you should have the certificates and kubeconfig file for each node. A list in the home folder should look like below:

Configuring the network

Get the POD_CIDR that will be used as part of network configuration

```
POD_CIDR=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^pod-cidr" | cut -d"=" -f2)
echo "${POD_CIDR}"
```

In case you are wondering where this POD_CIDR is coming from. Well, this was configured at the time of creating the worker nodes. Remember the for loop below? The --user-data flag is where we specified what we want the POD_CIDR to be. It is very important to ensure that the CIDR does not overlap with EC2 IPs within the subnet. In the real world, this will be decided in collaboration with the Network team.

Why do we need a network plugin? And why network configuration is crucial in implementing a Kubernetes cluster?

First, let us understand the Kubernetes networking model:

The networking model assumes a flat network, in which containers and nodes can communicate with each other. That means, regardless of which node is running the container in the cluster, Kubernetes expects that all the containers must be able to communicate with each other. Therefore, any network interface used for a Kubernetes implementation must follow this requirement. Otherwise, containers running in pods will not be able to communicate. Of course, this has security concerns. Because if an attacker is able to get into the cluster through a compromised container, then the entire cluster can be exploited.

To mitigate security risks and have a better controlled network topology, Kubernetes uses CNI (Container Network Interface) to manage Network Policies which can be used to operate the Pod network through external plugins such as Calico, Flannel or Weave Net to name a few. With these, you can set policies similar to how you would configure segurity groups in AWS and limit network communications through either cidr ipBlock, namespaceSelectors, or podSelectors, you will see more of these concepts further on.

To really understand Kubernetes further, let us explore some basic concepts around its networking:

Pods:

For example, if you deploy both Tooling and MySQL containers inside the same pod, then both of them are considered running on localhost. Although this design pattern is not ideal. Most likely they will run in separate Pods. In most cases one Pod contains just one container, but there are some design patterns that imply multi-container pods (e.g. sidecar, ambassador, adapter) – you can read more about them in this article.

For more detailed explanation of different aspects of Kubernetes networking – [Click here.](https://www.youtube.com/watch?v=5cNrTU6o3Fw)

Pod Network

You must decide on the Pod CIDR per worker node. Each worker node will run multiple pods, and each pod will have its own IP address. IP address of a particular Pod on worker node 1 should be able to communicate with the IP address of another particular Pod on worker node 2. For this to become possible, there must be a bridge network with virtual network interfaces that connects them all together. Here is an interesting read that goes a little deeper into how it works Bookmark that page and read it over and over again after you have completed this project


Configure the bridge and loopback networks Bridge

```
cat > 172-20-bridge.conf <<EOF
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```

Loopback

```
cat > 99-loopback.conf <<EOF
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF
```

Move the files to the network configuration directory

`$ sudo mv 172-20-bridge.conf 99-loopback.conf /etc/cni/net.d/`

Store the worker’s name in a variable

```
NAME=manual-k8s-cluster
WORKER_NAME=${NAME}-$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)
echo "${WORKER_NAME}"
```

![alt text](./images/57.png)

Move the certificates and kubeconfig file to their respective configuration directories

`$ sudo mv ${WORKER_NAME}-key.pem ${WORKER_NAME}.pem /var/lib/kubelet/`

`$ sudo mv ${WORKER_NAME}.kubeconfig /var/lib/kubelet/kubeconfig`

`$ sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig`

`$ sudo mv ca.pem /var/lib/kubernetes/`

Create the kubelet-config.yaml file

Ensure the needed variables exist

```
NAME=manual-k8s-cluster
WORKER_NAME=${NAME}-$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)
echo "${WORKER_NAME}"
```

![alt text](./images/58.png)

```
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
resolvConf: "/etc/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${WORKER_NAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${WORKER_NAME}-key.pem"
EOF
```

FINAL STEPS

Let us talk about the configuration file kubelet-config.yaml and the actual configuration for a bit. Before creating the systemd file for kubelet, it is recommended to create the kubelet-config.yaml and set the configuration there rather than using multiple startup flags in systemd. You will simply point to the yaml file.

The config file specifies where to find certificates, the DNS server, and authentication information. As you already know, kubelet is responsible for the containers running on the node, regardless if the runtime is Docker or Containerd; as long as the containers are being created through Kubernetes, kubelet manages them. If you run any docker or cri commands directly on a worker to create a container, bear in mind that Kubernetes is not aware of it, therefore kubelet will not manage those. Kubelet’s major responsibility is to always watch the containers in its care, by default every 20 seconds, and ensuring that they are always running. Think of it as a process watcher.

The clusterDNS is the address of the DNS server. As of Kubernetes v1.12, CoreDNS is the recommended DNS Server, hence we will go with that, rather than using legacy kube-dns.

Note: The CoreDNS Service is named kube-dns(When you see kube-dns, just know that it is using CoreDNS). This is more of a backward compatibility reasons for workloads that relied on the legacy kube-dns Service name.

In Kubernetes, Pods are able to find each other using service names through the internal DNS server. Every time a service is created, it gets registered in the DNS server.

In Linux, the /etc/resolv.conf file is where the DNS server is configured. If you want to use Google’s public DNS server (8.8.8.8) your /etc/resolv.conf file will have following entry:

nameserver 8.8.8.8

In Kubernetes, the kubelet process on a worker node configures each pod. Part of the configuration process is to create the file /etc/resolv.conf and specify the correct DNS server.

Configure the kubelet systemd service

```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service
[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --cluster-domain=cluster.local \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF
```

Create the kube-proxy.yaml file

```
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "172.31.0.0/16"
EOF
```

Configure the Kube Proxy systemd service

```
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes
[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF
```

Reload configurations and start both services

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable containerd kubelet kube-proxy
  sudo systemctl start containerd kubelet kube-proxy
}
```

Check status of the service

`$ sudo systemctl status containerd`

![alt text](./images/59.png)

<!-- {
  for i in 0 1 2; do
    instance="manual-k8s-cluster-worker-${i}"
    external_ip=$(aws ec2 describe-instances \
      --filters "Name=tag:Name,Values=${instance}" \
      --output text --query 'Reservations[].Instances[].PublicIpAddress')
    scp -i ssh/manual-k8s-cluster.id_rsa \
      manual-k8s-cluster-worker-0.kubeconfig ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
  done


  for i in 0 1 2; do
    instance="manual-k8s-cluster-worker-${i}"
    external_ip=$(aws ec2 describe-instances \
      --filters "Name=tag:Name,Values=${instance}" \
      --output text --query 'Reservations[].Instances[].PublicIpAddress')
    scp -i ssh/manual-k8s-cluster.id_rsa \
      manual-k8s-cluster-worker-1.kubeconfig ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
  done

  for i in 0 1 2; do
    instance="manual-k8s-cluster-worker-${i}"
    external_ip=$(aws ec2 describe-instances \
      --filters "Name=tag:Name,Values=${instance}" \
      --output text --query 'Reservations[].Instances[].PublicIpAddress')
    scp -i ssh/manual-k8s-cluster.id_rsa \
      manual-k8s-cluster-worker-2.kubeconfig ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
  done
} -->
