# Install Kubernetes with AWS Cloud Provider
Install Kubernetes (k8s) on AWS (Ubuntu 20.04 LTS Server)

# Setting VPC
Create new vpc with **10.0.0.0/16** CIDR

Name | Value
--- | ---
**Name tag** | K8s-cluster-vpc
**IPv4 CIDR block** | 10.0.0.0/16 
**IPv6 CIDR blocK** | No IPv6 CIDR block 
**TenancyInfo** | Default

Righ click, select **Manage tags** and add this tag on the previously created vpc

Key | Value
--- | ---
**Name** | K8s-cluster-vpc
**kubernetes.io/cluster/kubernetes** | owned

Select **Edit DNS hostnames** then enable DNS hostnames

Name | Value
--- | ---
**DNS hostnames** | ✅


# Subnet
Create a subnet for the vpc

Name | Value
--- | ---
**Name tag** | K8s-cluster-net
**VPC\*** | (Select K8s-cluster-vpc)
**Availability Zone** | (Select the desired zone E.g. **ap-southeast-1a**)
**VPC CIDRs** | (Auto generated)
**IPv4 CIDR block\*** | 10.0.0.0/24

Select **Modify auto-assign IP settings** then Enable Public IPs for EC2 instances 

Name | Value
--- | ---
**Auto-assign IPv4** | ✅ Enable auto-assign customer-owned IPv4 address

Select **Add/Edit Tags** then add this tag

Key | Value
--- | ---
**Name** | K8s-cluster-net
**kubernetes.io/cluster/kubernetes** | owned


# Internet Gateway
Create an Internet Gateway to route traffic from the subnet into the Internet

Name | Value
--- | ---
**Name tag** | K8s-cluster-igw

Select **Manage Tags** then add this tag

Key | Value
--- | ---
**Name** | k8s-cluster-igw
**kubernetes.io/cluster/kubernetes** | owned

Select **Attach to VPC** then attach to previously created vpc 

Name | Value
--- | ---
**VPC\*** | (Select K8s-cluster-vpc)


# Route Table
Create a routing tablle

Name | Value
--- | ---
**Name tag** | K8s-cluster-rtb
**VPC\*** | (Select K8s-cluster-vpc)

Select **Add/Edit Tags** then add this tag

Key | Value
--- | ---
**Name** | K8s-cluster-rtb
**kubernetes.io/cluster/kubernetes** | owned

Select **Edit routes** add a new route to the 0.0.0.0/0 network via the Internet Gateway we've created

Destination | Target | Status | Propagated
--- | --- | --- | --- 
10.0.0.0/16 | local | active | No
0.0.0.0/0 | (Select k8s-cluster-igw) | | No

Select **Edit subnet associations** then choose previously created subnet

\# | Subnet ID | IPv4 CIDR | IPv6 CIDR | Current Route Table
--- | --- | --- | --- | ---
✅ | (k8s-cluster-net) | 10.0.0.0/24 | - | Main


# IAM Role
Create IAM EC2 roles for node master and worker to make Kubernetes work on AWS

# IAM Master role
Create new Policy by Go to the **IAM -> Policies -> Create policy** then add this in to the JSON

Policy Name : **k8s-cluster-iam-master-policy**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeLaunchConfigurations",
                "autoscaling:DescribeTags",
                "ec2:DescribeInstances",
                "ec2:DescribeRegions",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVolumes",
                "ec2:CreateSecurityGroup",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:ModifyInstanceAttribute",
                "ec2:ModifyVolume",
                "ec2:AttachVolume",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:CreateRoute",
                "ec2:DeleteRoute",
                "ec2:DeleteSecurityGroup",
                "ec2:DeleteVolume",
                "ec2:DetachVolume",
                "ec2:RevokeSecurityGroupIngress",
                "ec2:DescribeVpcs",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:AttachLoadBalancerToSubnets",
                "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
                "elasticloadbalancing:CreateLoadBalancer",
                "elasticloadbalancing:CreateLoadBalancerPolicy",
                "elasticloadbalancing:CreateLoadBalancerListeners",
                "elasticloadbalancing:ConfigureHealthCheck",
                "elasticloadbalancing:DeleteLoadBalancer",
                "elasticloadbalancing:DeleteLoadBalancerListeners",
                "elasticloadbalancing:DescribeLoadBalancers",
                "elasticloadbalancing:DescribeLoadBalancerAttributes",
                "elasticloadbalancing:DetachLoadBalancerFromSubnets",
                "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                "elasticloadbalancing:ModifyLoadBalancerAttributes",
                "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
                "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:CreateTargetGroup",
                "elasticloadbalancing:DeleteListener",
                "elasticloadbalancing:DeleteTargetGroup",
                "elasticloadbalancing:DescribeListeners",
                "elasticloadbalancing:DescribeLoadBalancerPolicies",
                "elasticloadbalancing:DescribeTargetGroups",
                "elasticloadbalancing:DescribeTargetHealth",
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:ModifyTargetGroup",
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
                "iam:CreateServiceLinkedRole",
                "kms:DescribeKey"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Then go to **Roles** create new role with EC2 type
Search previously created policy (**k8s-cluster-iam-master-policy**)

Role Name: **k8s-cluster-iam-master-role**


# IAM Worker role
In the same way, Create new **Policy** then add this in to the JSON

Policy Name : **k8s-cluster-iam-worker-policy**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeInstances",
                "ec2:DescribeRegions",
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:BatchGetImage"
            ],
            "Resource": "*"
        }
    ]
}
```

Then go to **Roles** create new role with EC2 type
Search created policy (**k8s-cluster-iam-worker-policy**)

Role Name: **k8s-cluster-iam-worker-role**


# Running EC2
## Master Node
Create an EC2 Ubuntu Server 20.04 LTS using **t2.medium** type (*Recommended*). Kubernets master need at least 2 CPU cores, if you use Free Tear **t2.micro**, you'll facing error NumCPU on kubeadm init (It **can be ignores** but *Not Recommended*).

**# Choosing AMI**

Select **Ubuntu Server 20.04 LTS**

**# Choose Instance Type**

Select **t2.medium** 

**# Configure Instance Details**

Name | Value
--- | ---
**Network** | select previously created VPC (Select **K8s-cluster-vpc**)
**IAM role** | select previously created master role (Select **k8s-cluster-iam-master-role**)

**# Add Tags**

Key | Value
--- | ---
**Name** | Master
**kubernetes.io/cluster/kubernetes** | owned

**# Configure Security Group**

Type | Protocol | Port range | Source | Description - optional
--- | --- | --- | --- | ---
HTTP | TCP | 80 | 0.0.0.0/0 | Protocol HTTP
HTTPS | TCP | 443 | 0.0.0.0/0 | Protocol HTTPS
Custom TCP | TCP | 6443 | 0.0.0.0/0 | Kubernetes API Server
Custom TCP | TCP | 2379 - 2380 | 0.0.0.0/0 | etcd server client API
Custom TCP | TCP | 10250 | 0.0.0.0/0 | Kubelet API
Custom TCP | TCP | 10251 | 0.0.0.0/0 | kube-scheduler
Custom TCP | TCP | 10252 | 0.0.0.0/0 | kube-controller-manager
Custom TCP | TCP | 10255 | 0.0.0.0/0 | Read-Only Kubelet API
Custom TCP | TCP | 10259 | 0.0.0.0/0 | Schedular
Custom TCP | TCP | 10257 | 0.0.0.0/0 | Controller
All Traffic | All | All | 10.0.0.0/16 | VPC subnet

Review all changes then **Launch**

## Worker Node
While waiting master node, create new EC2 instane in the same way like the master node but using **k8s-cluster-iam-worker-rolee**

**# Choosing AMI**

Select **Ubuntu Server 20.04 LTS**

**# Choose Instance Type**

Select **t2.medium** 

**# Configure Instance Details**

Name | Value
--- | ---
**Network** | select previously created VPC (Select **K8s-cluster-vpc**)
**IAM role** | select previously created master role (Select **k8s-cluster-iam-worker-role**)

**# Add Tags**

Key | Value
--- | ---
**Name** | Worker
**kubernetes.io/cluster/kubernetes** | owned

**# Configure Security Group**

Type | Protocol | Port range | Source | Description - optional
--- | --- | --- | --- | ---
SSH | TCP | 22 | 0.0.0.0/0 | SSH
HTTP | TCP | 80 | 0.0.0.0/0 | Protocol HTTP
HTTPS | TCP | 443 | 0.0.0.0/0 | Protocol HTTPS
Custom TCP | TCP | 10250 | 0.0.0.0/0 | Kubelet API
Custom TCP | TCP | 10255 | 0.0.0.0/0 | Read-Only Kubelet API
Custom TCP | TCP | 30000-32767 | 0.0.0.0/0 | NodePort Services
All Traffic | All | All | 10.0.0.0/16 | VPC subnet

Review all changes then **Launch**

# Config Kubernetes Cluster
## Master Node Setup

```bash
sudo su -
# upgrade & install depedencies
apt update && apt upgrade -y && apt dist-upgrade -y
apt install -y apt-transport-https ca-certificates curl software-properties-common gnupg2 net-tools

# set FQDN hostname
hostnamectl set-hostname $(curl -s http://169.254.169.254/latest/meta-data/local-hostname)

# install docker
apt install -y docker.io
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
systemctl stop docker
systemctl start docker
systemctl enable docker

cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --system

# install kubelet kubeadm kubectl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
apt update
apt install -y kubelet kubeadm kubectl

cat << EOF > /etc/kubernetes/aws.yaml
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
networking:
  serviceSubnet: "10.100.0.0/16"
  podSubnet: "10.244.0.0/16"
apiServer:
  extraArgs:
    cloud-provider: "aws"
controllerManager:
  extraArgs:
    cloud-provider: "aws"
EOF

# init cluster
kubeadm init --config /etc/kubernetes/aws.yaml

```
**NB**: if you use **Free tier t2.micro**, add argument **--ignore-preflight-errors=NumCPU**
`kubeadm init --config /etc/kubernetes/aws.yaml --ignore-preflight-errors=NumCPU`

After cluster init, you'll get unique token for worker node to join:

kubeadm join <span style="color:red">10.0.0.119:6443</span> --token <span style="color:red">i4pna1.8tlp6kcmukr5sian</span> --discovery-token-ca-cert-hash <span style="color:red">sha256:c2974f5f46e06df9bddd532ac61617ada82943b09ee914847fd8f15f7b8ff008</span>

save it, we'll use later in worker node

```bash
# save kube config
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# install Flannel CNI
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl get nodes

```

## Worker Node Setup
