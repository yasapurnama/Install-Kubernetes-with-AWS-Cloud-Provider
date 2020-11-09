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

Righ click, select **Edit DNS hostnames** then enable DNS hostnames

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

Righ click, select **Modify auto-assign IP settings** then Enable Public IPs for EC2 instances 

Name | Value
--- | ---
**Auto-assign IPv4** | ✅ Enable auto-assign customer-owned IPv4 address

Righ click, select **Add/Edit Tags** then add this tag

Key | Value
--- | ---
**Name** | K8s-cluster-net
**kubernetes.io/cluster/kubernetes** | owned


# Internet Gateway
Create an Internet Gateway to route traffic from the subnet into the Internet

Name | Value
--- | ---
**Name tag** | K8s-cluster-igw

Righ click, select **Manage Tags** then add this tag

Key | Value
--- | ---
**Name** | k8s-cluster-igw
**kubernetes.io/cluster/kubernetes** | owned

Righ click, select **Attach to VPC** then attach to previously created vpc 

Name | Value
--- | ---
**VPC\*** | (Select K8s-cluster-vpc)


# Route Table
Create a routing tablle

Name | Value
--- | ---
**Name tag** | K8s-cluster-rtb
**VPC\*** | (Select K8s-cluster-vpc)

Righ click, select **Add/Edit Tags** then add this tag

Key | Value
--- | ---
**Name** | K8s-cluster-rtb
**kubernetes.io/cluster/kubernetes** | owned

Righ click, select **Edit routes** add a new route to the 0.0.0.0/0 network via the Internet Gateway we've created

Destination | Target | Status | Propagated
--- | --- | --- | --- 
10.0.0.0/16 | local | active | No
0.0.0.0/0 | (Select k8s-cluster-igw) | | No

Righ click, select **Edit subnet associations** then choose previously created subnet

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