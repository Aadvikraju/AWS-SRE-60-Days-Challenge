# Day 3 - IAM, EC2 & Security Group Fundamentals

## 1. Theory - SRE Core Concepts

### **IAM - Identity & Access Management**
IAM is AWS security guard. It controls "Who can do What" in AWS account.

**3 Pillars:**
1. **Authentication** = Who are you? → Password, MFA
2. **Authorization** = What can you do? → Policies, Roles  
3. **Audit** = What did you do? → CloudTrail logs

**Interview Q**: Difference between Root user and IAM User?  
**Answer**: Root user = Master key of house. IAM user = Separate keys for each room. Using root daily is a security risk.

### **MFA - Multi Factor Authentication**
Extra 6-digit code after password from phone app. 
**SRE Rule**: MFA is mandatory on production accounts. 90% of AWS hacks happen due to no MFA.

### **EC2 - Elastic Compute Cloud**
AWS virtual server. `t3.micro` = 1 vCPU, 1GB RAM. Comes under AWS Free Tier.
**Region ap-south-1 Mumbai**: Lower latency for India users + GST billing.

### **Security Group - Virtual Firewall**
Acts like firewall for EC2 instance. Only allowed traffic can enter.

**Key Rules**:
- SG is Stateful. If inbound SSH is allowed, outbound reply is auto-allowed.
- Least Privilege: Never use `0.0.0.0/0`. Use `My IP/32` for SSH only.

**Interview Q**: Difference between Security Group and NACL?  
**Answer**: SG = Instance level, stateful. NACL = Subnet level, stateless.

### **IAM Role - Keyless Authentication**
To give EC2 access to S3, 2 methods:
1. **Wrong**: Put Access Keys in EC2 → If leaked, full account compromise
2. **Correct**: Attach IAM Role → AWS provides temporary credentials automatically

**Trust Policy**: Defines which AWS service can assume this role. For EC2: `"Service": "ec2.amazonaws.com"`

---

## 2. Hands-on Lab - Step by Step

### **Task 1: Create IAM User + Enable MFA**
1. IAM → Users → Create user: `Day1-task`
2. Select AWS Management Console access
3. Attach policy: `AdministratorAccess`  
   *Note: For production use ReadOnlyAccess. Admin used here for learning only.*
4. After creation → Security credentials → MFA → Assign MFA device → Use Authenticator app

### **Task 2: Launch EC2 Instance**
1. EC2 → Launch instance
2. AMI: Amazon Linux 2023 Free Tier eligible
3. Instance type: `t3.micro`
4. Key pair: Create `raju-key.pem` and download
5. Network: Default VPC, Default Subnet
6. Security Group: Create new SG → Name `Raju-SSH-SG`

### **Task 3: Configure Security Group for SSH**
Edit inbound rules of `Raju-SSH-SG`:Type: SSH
Protocol: TCPPort: 22
Source: My IP /32
Description: Allow SSH from my IP onlyjavascript**SRE Tip**: Never use `0.0.0.0/0` for SSH port 22. It will attract brute force attacks.

### **Task 4: Create IAM Role and Attach to EC2**
1. IAM → Roles → Create role
2. Trusted entity: AWS service → EC2
3. Permissions: `AmazonEC2ReadOnlyAccess` + `AmazonS3ReadOnlyAccess`
4. Role name: `EC2-S3-ReadOnly-Role`

**Attach Role to EC2:**
EC2 → Select instance → Actions → Security → Modify IAM role → Select `EC2-S3-ReadOnly-Role`

**Test from EC2 SSH:**
```bash
sudo dnf install awscli -y
aws --version
aws s3 lsResult: S3 bucket list appears without running aws configure ✅ 
This proves IAM Role is working and providing credentials.3. Key SRE Learnings from Day 3
Never use root account: Use IAM user + MFA for daily workLeast Privilege: Allow SSH only from your IP. Deny everything elseNo Hardcoded Keys: Use IAM Role for EC2 to AWS service access, not Access KeysRegion Selection: Choose region close to users for low latency and data compliance4. Common Mistakes & FixesMistakeImpactFixSG: 0.0.0.0/0 for SSH port 22Server will get hackedUse My IP/32Storing Access Keys on EC2Key leak = Full account accessUse IAM RoleNo MFA enabledPassword leak = Full accessEnable MFA mandatoryScreenshots for GitHub Repo:
IAM User with MFA Enabled statusEC2 Instance details showing t3.micro + ap-south-1 regionSecurity Group Inbound rule with My IP/32IAM Role attached to EC2 instanceaws s3 ls command output from EC2 terminal
