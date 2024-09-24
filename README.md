# Guidance for Oracle Migrations to Amazon Aurora PostgreSQL using AWS DMS 

## Table of Contents

List the top-level sections of the README template, along with a hyperlink to the specific section.

1. [Overview](#overview-required)
    - [Cost](#cost)
2. [Prerequisites](#prerequisites-required)
    - [Operating System](#operating-system-required)
3. [Deployment Steps](#deployment-steps-required)
4. [Deployment Validation](#deployment-validation-required)
5. [Running the Guidance](#running-the-guidance-required)
6. [Next Steps](#next-steps-required)
7. [Cleanup](#cleanup-required)

***Optional***

8. [FAQ, known issues, additional considerations, and limitations](#faq-known-issues-additional-considerations-and-limitations-optional)
9. [Revisions](#revisions-optional)
10. [Notices](#notices-optional)
11. [Authors](#authors-optional)

## Overview

This guidance demonstrates how to efficiently migrate very large tables from Oracle to Amazon Aurora PostgreSQL. The guidance leverages multiple DMS tasks that have been preconfigured for high performance for both the full load and CDC stages of the migration. Users can further tune DMS parameters for their particular workload, but this provides a very good starting point for large databases. Included sample code contains a Cloudformation template that builds out the entire environment, including the Oracle database, DMS Infrastructure, DMS tasks, and Aurora PostgreSQL database.

![Architecture diagram](./assets/images/Arch_Diagram.png)


### Cost

You are responsible for the cost of the AWS services used while running this Guidance.

As of 09/09/2024, the cost for running this guidance with the default settings in the US-West-2 (Oregon) is approximately $19,596.97 per month.

The cloudformation template takes inputs on the size of instances used, so this can be scaled up or down as needed. It also does not need to be run for a full month to understand the guidance and apply it to your particular workload.

We recommend creating a Budget through AWS Cost Explorer to help manage costs. Prices are subject to change. For full details, refer to the pricing webpage for each AWS service used in this Guidance.

The following table provides a sample cost breakdown for deploying this Guidance with the default parameters in the US-West-2 (Oregon) Region for one month.

| AWS service  | Dimensions | Monthly Cost [USD] |
| ----------- | ------------ | ------------ |
| Amazon RDS for Oracle | 16.xl and 5 TB Storage | $6,017.88 |
| Amazon Aurora PostgreSQL | 16.xl and 5 TB Storage | $7,729.01 |
| DMS Replication Instance | 16.xl and 500 GB Storage | 5,326.11 | 
| Amazon EC2 | m5.4xl | 523.97 | 

## Prerequisites 

- Access to deploy Cloudformation template and create resources (Amazon EC2, Aurora PostgreSQL, RDS Oracle, VPC, Subnets, Security groups, IAM roles and Policies)

### Operating System 

- These deployment instructions are optimized to best work on **<Amazon Linux 2023 AMI>** but might work on any linux or Mac environment with little modification.


### AWS account requirements 

This deployment requires that you have access to the following AWS services:

- Amazon RDS
- Amazon EC2
- Amazon S3
- Amazon VPC
- AWS IAM  
- AWS Cloudformation

### Supported Regions 

This guidance is supported in all AWS Regions


## Deployment Steps


1. Clone the repo using command
```
git clone https://github.com/aws-solutions-library-samples/guidance-for-large-data-migrations-from-oracle-to-amazon-aurora-postgresql
```

2. Deploy the Cloudformation template

    Using the AWS Management Console

    For this guidance, we will be using the us-west-2 region.
    Sign in to the AWS CloudFormation console.
   
    Click Create Stack and upload the oracletoaurorapgv*.yml file.
   
    Deploy the stack after entering dms-solution in the stack name.
        The parameters can be changed as desired but the solution has been tested and verified with the defaults.
4. Open CloudFormation console and verify the status of the template with the name starting with dms-solution*.
   You should also see an Oracle and Aurora PostgreSQL database under the RDS section of the management console.

   ![Validation Example](./assets/images/validation.png)


## Running the Guidance (required)

1. Login to the EC2 instance using Systems Manager (EC2 page -> Connect -> SSM)
   
2. Change Directory

```
cd /home/dms
```

3. Clone the repo to the bastion host. We are using a Bastion to increase security since the database resides in a private VPC.
   
```
git clone https://github.com/aws-solutions-library-samples/guidance-for-large-data-migrations-from-oracle-to-amazon-aurora-postgresql
```


4. Configure the environment

```
sh export_variables.sh
```

5. Source environment variables

```
source ~/.bashrc
```

6. Configure Source DB

```
sh oracle_prerequisites_dms.sh
```

7. Download Oracle tools software
   
```
wget https://download.oracle.com/otn_software/linux/instantclient/2115000/instantclient-tools-linux.x64-21.15.0.0.0dbru.zip
```

8. Install unzip

```
sudo yum install unzip
```

9. Unzip software

```
unzip instantclient-tools-linux.x64-21.15.0.0.0dbru.zip
```

10. Add instant client to the PATH environmment variable
```
export PATH=$PATH:/home/ec2-user/instantclient_21_15/
```
11. Change directory into the repo

```cd guidance*
```

12. Connect to Oracle. Run the procedure to build the Schema and load the inital data set. 

```
sh oracle_connect.sh
```

```
@BuildAndLoadSchema.sql
```


12. Login to the DMS section of the AWS Console. Under Migration Tasks you will see 4 examples. The tasks are configured for optimal performance for each of the given scenarios
1/ non-parallel loading a table 2/ parallel loading a partitioned table 3/ parallel loading a large table that isn't partitioned using boundary ranges and 4/ parallel loading a table with subpartitions (this is the largest table). Run the task(s) of your choice and evaluate performance. We discuss configuration settings in the next section.

13. Choose one of the DMS tasks and run it. Multiple tasks could run in parallel but will affect performance.


## Next Steps (required)

14. Monitor the DMS task performance in the AWS CloudWatch Console.



In this guidance we have modified the maxFileSize and maxFullLoadSubTasks values to take advantage of the resources of the relatively large instances sizes we are using. These values can be further tuned for your specific workload, and to put more or less stress on the source database. Instance sizes can also be reduced or increased. If lowering the instance class of the DMS replication instance we recommend leaving the 1TB of storage for more IOPS and throughput of that volume. 


## Cleanup (required)

Delete the Cloudformation stack will remove all resources. 


## FAQ, known issues, additional considerations, and limitations (optional)


**Known issues (optional)**

Using maxFullLoadSubTasks value of 49 can lead to a race condition which may cause the task to fail. 


## Notices

*Customers are responsible for making their own independent assessment of the information in this Guidance. This Guidance: (a) is for informational purposes only, (b) represents AWS current product offerings and practices, which are subject to change without notice, and (c) does not create any commitments or assurances from AWS and its affiliates, suppliers or licensors. AWS products or services are provided “as is” without warranties, representations, or conditions of any kind, whether express or implied. AWS responsibilities and liabilities to its customers are controlled by AWS agreements, and this Guidance is not part of, nor does it modify, any agreement between AWS and its customers.*


## Authors 

Name of code contributors
