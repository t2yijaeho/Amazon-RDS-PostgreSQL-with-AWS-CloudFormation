# Amazon RDS PostgreSQL with AWS CloudFormation


## 1. Launch AWS CloudShell

1. Sign in to AWS Management Console <img src="https://github.com/t2yijaeho/Amazon-RDS-PostgreSQL-with-AWS-CloudFormation/blob/matia/images/AWS%20Management%20Console.png?raw=true" width="16">

2. Choose the AWS CloudShell icon <img src="https://github.com/t2yijaeho/Amazon-RDS-PostgreSQL-with-AWS-CloudFormation/blob/matia/images/AWS%20CloudShell.png?raw=true" width="16"> on the navigation bar

3. Check AWS Command Line Interface (AWS CLI) version

    ```bash
    aws --version
    ```

    <img src="https://github.com/t2yijaeho/Amazon-RDS-PostgreSQL-with-AWS-CloudFormation/blob/matia/images/AWS%20CloudShell%20version.png?raw=true">


## 2. Create an Amazon RDS PostgreSQL

1. Get an AWS CloudFormation stack template body

    ```bash
    wget https://github.com/t2yijaeho/Amazon-RDS-PostgreSQL-with-AWS-CloudFormation/raw/matia/Template/RDS-PostgreSQL.yaml
    ```


2. Create an AWS CloudFormation stack

    ```bash
    aws cloudformation create-stack \
      --stack-name RDS-PostgreSQL \
      --template-body file://./RDS-PostgreSQL.yaml
    ```

3. AWS CloudFormation returns following output

    ```json
    {
    "StackId": "arn:aws:cloudformation:us-abcd-x:123456789012:stack/RDS-PostgreSQL/b4d0f5e0-d4c2-11ec-9529-06edcc65f112"
    }
    ```

4. Monitor the progress by the stack's events in AWS management console

    <img src="https://github.com/t2yijaeho/Amazon-RDS-PostgreSQL-with-AWS-CloudFormation/blob/matia/images/CloudFormation%20Stack%20Creation%20Events.png?raw=true">


## 3. Install PostgreSQL Client

1. List the available PostgreSQL toics from the Extras Library

    ```bash
    sudo amazon-linux-extras | grep postgresql
    ```

2. Enable the desired latest PostgreSQL topic

    ```bash
    sudo amazon-linux-extras enable postgresql13 | grep postgresql
    ```

2. Install PostgreSQL topic

    ```bash
    sudo yum clean metadata && sudo yum install -y postgresql
    ```

3. Verify the installation and confirm the PostgreSQL Client version:

    ```bash
    sudo yum list installed postgresql
    ```
    ```bash
    psql --version
    ```


## 4. Connect to an Amazon RDS PostgreSQL

1. Add inboud rule to Amazon RDS PostgreSQL Security Group

    Find CloudShell IP address
    ```bash
    CLOUDSHELL_IP_ADDRESS=$(curl https://checkip.amazonaws.com)
    echo $CLOUDSHELL_IP_ADDRESS
    ```
    
    Find Amazon RDS PostgreSQL Security Group ID
    ```bash
    RDS_SECURITY_GROUP_ID=$(aws rds describe-db-instances \
        --db-instance-identifier targetdb \
      | jq --raw-output \
        ".DBInstances[0].VpcSecurityGroups[0].VpcSecurityGroupId")
    echo $RDS_SECURITY_GROUP_ID
    ```
    
    Add CloudShell IP address to Security Group inboud rule
    ```bash
    aws ec2 authorize-security-group-ingress \
      --group-id $RDS_SECURITY_GROUP_ID \
      --protocol tcp --port 45432 \
      --cidr "${CLOUDSHELL_IP_ADDRESS}/32"
    ```

2. Describe Amazon RDS PostgreSQL

    Describe Amazon RDS PostgreSQL instance 
    ```bash
    aws rds describe-db-instances \
        --db-instance-identifier targetdb
    ```
    
    
    ```bash
    aws rds describe-db-instances \
        --db-instance-identifier targetdb \
      | jq --raw-output \
        "(\"Endpoint : \" + .DBInstances[0].Endpoint.Address), (\"Port : \" + (.DBInstances[0].Endpoint.Port | tostring)), (\"Master Username : \" + .DBInstances[0].MasterUsername), (\"DB Name : \" + .DBInstances[0].DBName)"
    ```
    
    Put DB instance endpoint address to a variable
    ```bash
    DB_INSTANCE_ENDPOINT=$( \
      aws rds describe-db-instances \
        --db-instance-identifier targetdb \
      | jq --raw-output \
        '.DBInstances[0].Endpoint.Address')
    echo $DB_INSTANCE_ENDPOINT
    ```

3. Connect to Amazon RDS PostgreSQL instance using PostgreSQL Client 

    Save Password to .pgpass file
    ```bash
    (umask 177 ; \
      echo $DB_INSTANCE_ENDPOINT:45432:postgres:postgres:target1234 > .pgpass)
    ```

    Connect to instance
    ```bash
    psql \
      --host=$DB_INSTANCE_ENDPOINT \
      --port=45432 \
      --username=postgres \
      --no-password \
      --dbname=postgres
    ```
    
    PostgreSQL Client provides a prompt with the name of the database
    ```bash
    psql (13.3, server 13.4)
    SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
    Type "help" for help.

    postgres=>
    ```

