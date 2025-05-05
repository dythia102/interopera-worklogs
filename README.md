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
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text > my-key.pem
chmod 400 my-key.pem
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
    --group-names my-sg \
    --region ap-southeast-1 \
    --query "SecurityGroups[*].IpPermissions[*].[IpProtocol,FromPort,ToPort,IpRanges[*].CidrIp]" \
    --output table
```

---
# TODO
### **5. Launch the EC2 Instance**
```bash
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \ # Replace with real AMI ID
  --count 1 \
  --instance-type t2.micro \
  --key-name my-key \
  --security-groups my-sg
```

Output will show `InstanceId`, e.g., `i-0abcd12345efgh678`.

---

### **6. Allocate and Associate Elastic IP (optional, for static IP)**
```bash
aws ec2 allocate-address
aws ec2 associate-address --instance-id i-0abcd12345efgh678 --allocation-id eipalloc-xxxxxxxx
```

---

### **7. Get Public IP**
```bash
aws ec2 describe-instances --instance-ids i-0abcd12345efgh678 \
  --query "Reservations[*].Instances[*].PublicIpAddress" --output text
```

---

### **8. SSH into your instance**
```bash
ssh -i my-key.pem ec2-user@<PUBLIC-IP>   # Amazon Linux
# or
ssh -i my-key.pem ubuntu@<PUBLIC-IP>     # Ubuntu
```

---

Let me know if you want a **bash script** version of this setup.
