# **Work Logs** 
launch and configure an **EC2 instance using AWS CLI**

---

## ✅ Step-by-step EC2 Setup via AWS CLI

### **0. Prerequisites**
- [x] AWS CLI installed (`aws --version`)
- [x] Run `aws configure` with your:
  - `AWS Access Key ID`
  - `AWS Secret Access Key`
  - `Default region name` (e.g., `ap-southeast-1`)
  - `Default output format` (e.g., `json`)
  #### Verification 
  ```bash
    aws --version        # Should return aws-cli version
    aws configure list   # Should show your configured Access Key, Region, etc.
  ```
---

### **1. Choose an AMI**
Use Amazon Linux 2023 with ARM Arch, for price peformance:

  ```bash
  aws ec2 describe-images \
    --owners amazon \
    --filters "Name=name,Values=al2023-ami-*" \
              "Name=architecture,Values=arm64" \
              "Name=virtualization-type,Values=hvm" \
              "Name=root-device-type,Values=ebs" \
    --region ap-southeast-1 \
    --query "sort_by(Images, &CreationDate)[::-1].[ImageId, Name, CreationDate]" \
    --output table
  ```
  will using this image
  - ami-0d47fa2c431cf6d45
  - al2023-ami-ecs-hvm-2023.0.20250430-kernel-6.1-arm64
  - 2025-04-30T21:27:09.000Z
---

### **2. Create a Key Pair (for SSH access)**
```bash
aws ec2 create-key-pair --key-name aws-interopera --query 'KeyMaterial' --output text > aws-interopera.awskeypair.pem
chmod 400 aws-interopera.awskeypair.pem
```
#### Verification:
  ```bash
  aws ec2 describe-key-pairs --query "KeyPairs[*].KeyName" --output table
  ```
---

### **3. Create a Security Group**
  ```bash
  aws ec2 create-security-group --group-name aws-interopera-secgroup --description "Allow SSH and web traffic"
  ```
#### Verification:
  ```bash
  aws ec2 describe-security-groups \
  --region ap-southeast-1 \
  --query "SecurityGroups[*].[GroupName, GroupId, Description]" \
  --output table
  ```
---
### **4. Allow Ports (SSH, HTTP, HTTPS, custom)**
```bash
aws ec2 authorize-security-group-ingress --group-name aws-interopera-secgroup --protocol tcp --port 22 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-name aws-interopera-secgroup --protocol tcp --port 80 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-name aws-interopera-secgroup --protocol tcp --port 443 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-name aws-interopera-secgroup --protocol tcp --port 8000 --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress --group-name aws-interopera-secgroup --protocol tcp --port 3000 --cidr 0.0.0.0/0
```
#### Verification:
```bash
  aws ec2 describe-security-groups \
    --group-names aws-interopera-secgroup \
    --region ap-southeast-1 \
    --query "SecurityGroups[*].IpPermissions[*].[IpProtocol,FromPort,ToPort,IpRanges[*].CidrIp]" \
    --output table
```

---
### **5. Launch the EC2 Instance**
```bash
aws ec2 run-instances --image-id ami-0d47fa2c431cf6d45 --count 1 --instance-type t4g.micro --key-name aws-interopera --security-groups aws-interopera-secgroup --region ap-southeast-1
```

#### OUTPUT
```json
{
    "ReservationId": "r-09e6595b30c8da164",
    "OwnerId": "094604222284",
    "Groups": [],
    "Instances": [
        {
            "Architecture": "arm64",
            "BlockDeviceMappings": [],
            "ClientToken": "3c8474b6-dbaa-450a-a61d-929b0692c7ae",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2025-05-05T20:19:20+00:00",
                        "AttachmentId": "eni-attach-0ab0c91c8a8e0d4f2",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupId": "sg-0226dc0fa229f050a",
                            "GroupName": "aws-interopera-secgroup"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "06:e2:75:77:b2:27",
                    "NetworkInterfaceId": "eni-012749e75ce295830",
                    "OwnerId": "094604222284",
                    "PrivateDnsName": "ip-172-31-44-94.ap-southeast-1.compute.internal",
                    "PrivateIpAddress": "172.31.44.94",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-44-94.ap-southeast-1.compute.internal",
                            "PrivateIpAddress": "172.31.44.94"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0ce769afbe5db3ee7",
                    "VpcId": "vpc-08ff9d8e3a93ce8a0",
                    "InterfaceType": "interface",
                    "Operator": {
                        "Managed": false
                    }
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupId": "sg-0226dc0fa229f050a",
                    "GroupName": "aws-interopera-secgroup"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 2,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "required",
                "HttpPutResponseHopLimit": 2,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
            "BootMode": "uefi",
            "PrivateDnsNameOptions": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            },
            "MaintenanceOptions": {
                "AutoRecovery": "default"
            },
            "CurrentInstanceBootMode": "uefi",
            "Operator": {
                "Managed": false
            },
            "InstanceId": "i-085ae187d97a893f2",
            "ImageId": "ami-0d47fa2c431cf6d45",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "PrivateDnsName": "ip-172-31-44-94.ap-southeast-1.compute.internal",
            "PublicDnsName": "",
            "StateTransitionReason": "",
            "KeyName": "aws-interopera",
            "AmiLaunchIndex": 0,
            "ProductCodes": [],
            "InstanceType": "t4g.micro",
            "LaunchTime": "2025-05-05T20:19:20+00:00",
            "Placement": {
                "GroupName": "",
                "Tenancy": "default",
                "AvailabilityZone": "ap-southeast-1b"
            },
            "Monitoring": {
                "State": "disabled"
            },
            "SubnetId": "subnet-0ce769afbe5db3ee7",
            "VpcId": "vpc-08ff9d8e3a93ce8a0",
            "PrivateIpAddress": "172.31.44.94"
        }
    ]
}
```

#### Verification:
    ```bash
    aws ec2 describe-instances \
    --region ap-southeast-1 \
    --query "Reservations[*].Instances[*].[InstanceId, State.Name, PublicIpAddress]" \
    --output table
    ```

#### Public IP:
    ```bash
    aws ec2 describe-instances \
    --instance-ids i-085ae187d97a893f2 \
    --region ap-southeast-1 \
    --query "Reservations[*].Instances[*].PublicIpAddress" \
    --output text
    ```bash

---

### **6. Allocate and Associate Elastic IP (optional, for static IP)**
    ```bash
    aws ec2 allocate-address
    aws ec2 associate-address --instance-id i-085ae187d97a893f2 --allocation-id eipalloc-0e7cbc1babeffec97
    ```

    ```json
    {
        "AllocationId": "eipalloc-0e7cbc1babeffec97",
        "PublicIpv4Pool": "amazon",
        "NetworkBorderGroup": "ap-southeast-1",
        "Domain": "vpc",
        "PublicIp": "52.74.52.230"
    }
    ```

    ```json
    {
        "AssociationId": "eipassoc-08a6d4e387dee5aa9"
    }
    ```
---

### **8. SSH into your instance**
    ```bash
    ssh -i aws-interopera.awskeypair.pem ec2-user@13.212.186.153
    ```



---

Let me know if you want a **bash script** version of this setup.
