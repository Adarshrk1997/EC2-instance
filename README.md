AWS-CLI-EC2-instance

Launching an EC2 instance using AWS CLI

#**Features**:
 
Security group to allow ports 22,80,443.
Key pair for this instance.

#**Step 1: Configuring IAM user** 

Created an IAM user with adminstrator access to manage AWS account throuh aws_cli.

```
> aws configure
AWS Access Key ID [None]: *************
AWS Secret Access Key [None]: *************
Default region name [None]: ap-south-1 <<Region
Default output format [None]: json
```
#**Step 2: Creating keypair for this instance**

New key pair is created and the private key is saved to main-key.pem.

```
  aws ec2 create-key-pair --key-name main-key --query "KeyMaterial" --output text > main-key.pem

  [root@ip-172-31-46-214 ~]# ls -l

-rw-r--r-- 1 root root 1675 Aug  9 18:14 main-key.pem
```

To describe the new Keypair :


```
  [root@ip-172-31-46-214 ~]# aws ec2 describe-key-pairs --key-name main-key
{
    "KeyPairs": [
        {
            "Tags": [], 
            "KeyName": "main-key", 
            "KeyFingerprint": "7c:2b:cf:aa:d8:7d:a7:20:b6:4b:05:67:1b:77:47:c3:47:08:79:06", 
            "KeyPairId": "key-021331cd4f8386f8d"
        }
    ]
}
```
#**Step 3: Creating a new security group**

To create a new security group with name "main-sg"
```
  > aws ec2 create-security-group --group-name main-sg  --description "Allow 22,80,443"
{
    "GroupId": "sg-036ce110e6b615a18"
}
```
Adding tag to the new security group "main-sg"

```
> aws ec2 create-tags --resources sg-036ce110e6b615a18 --tags Key=Name,Value=main-sg
```

Enabling 22,80,443 ports on the new security group

```
# aws ec2 authorize-security-group-ingress --group-name main-sg --protocol tcp --port 22 --cidr 0.0.0.0/0

# aws ec2 authorize-security-group-ingress --group-name main-sg  --protocol tcp --port 80 --cidr 0.0.0.0/0

# aws ec2 authorize-security-group-ingress --group-name main-sg  --protocol tcp --port 443 --cidr 0.0.0.0/0
```

Cross checking the security group informations

```
> aws  ec2  describe-security-groups --group-name main-sg
```

**Step 4: Creating a new instance for my webserver
**

```
 [root@ip-172-31-46-214 ~]# aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --region ap-south-1
{
    "InvalidParameters": [], 
    "Parameters": [
        {
            "Name": "/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2", 
            "DataType": "text", 
            "LastModifiedDate": 1659375900.74, 
            "Value": "ami-00c5a35740f4561e7", 
            "Version": 71, 
            "Type": "String", 
            "ARN": "arn:aws:ssm:ap-south-1::parameter/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2"
        }
    ]

```

```
  aws ec2 run-instances --image-id ami-00c5a35740f4561e7 --security-group-ids sg-036ce110e6b615a18 --instance-type t2.micro --key-name main-key
```
~```

[root@ip-172-31-46-214 ~]# aws ec2 create-tags --resources i-01bdad45da04f638b --tags Key=Name,Value=website-main
```
```
  [root@ip-172-31-46-214 ~]# aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro" --query "Reservations[].Instances[].InstanceId"


[


    "i-01bdad45da04f638b"

]
```
