# OpenData Sequence Read Archive - AWS Data Science Exploration

**Important Notice:** These application solutions are not supported products in their own right, but examples to help our customers use our products from their applications. As our customer, any applications you integrate these examples in should be thoroughly tested, secured, and optimized according to your business's security standards before deploying to production or handling production workloads.

Jupyter notebooks designed to help data scientists explore AWS technologies using genomics datasets from the Sequence Read Archive (SRA).

## Overview

This project provides a hands-on learning environment for data scientists to:
- Work with real genomics data from NCBI's Sequence Read Archive
- Download SRA data directly from AWS Open Data Registry
- Perform comprehensive quality control analysis (FastQC-style)
- Learn cloud-based bioinformatics workflows on AWS
- Understand sequencing data quality metrics

## Files

- **`opendata_sra_quality_analysis.ipynb`** - Comprehensive SRA quality analysis notebook
  - Search and discover SRA datasets using metadata queries
  - Download .sra files from AWS S3 (Open Data Registry)
  - Convert SRA to FASTQ format
  - Perform 7 FastQC-style quality control metrics
  - Generate quality reports with Pass/Warn/Fail indicators
  - Optimized for ml.t3.medium instance

- **`environment.yml`** - CloudFormation template to deploy SageMaker infrastructure

## Deployment Instructions

### 1. Deploy the Infrastructure
Deploy the AWS CloudFormation template to create the Amazon SageMaker environment:

```bash
aws cloudformation create-stack \
  --stack-name opendata-sra-notebook-stack \
  --template-body file://environment.yml \
  --capabilities CAPABILITY_IAM \
  --region us-east-1
```

### 2. Access the Notebook
Once the stack is deployed (5-10 minutes):

1. **Go to Amazon SageMaker AI** in the AWS Console
2. **From the left-hand menu, choose "Notebooks"** to find the notebook instances deployed by the CloudFormation template
3. **Select the notebook instance** named `opendata-sra-notebook`
4. **Click "Open Jupyter"** to launch the Jupyter environment
5. **Navigate to the project folder** `notebooks/aws-ncbi-sra` containing the notebook
6. **Open `opendata_sra_quality_analysis.ipynb`**
7. **Select the default Kernel** and click Select
8. **Update the S3 dest_bucket name** `Part 1: Configure AWS S3 Storage` in the notebook cell (bucket name found in CloudFormation Outputs tab)
9. **Run the cells** following the instructions in the notebook

### 3. What to Expect

**With the Quality Analysis Notebook:**
- Installation: 3-5 minutes
- Download (7MB dataset): 30 seconds
- Conversion: 1-2 minutes
- Quality Analysis: 2-3 minutes
- **Total Runtime: ~10 minutes**
- **Instance**: ml.t3.medium works great

## Cost Estimation

### Expected 24-Hour Costs (US East 1):
**Total Estimated Cost: ~$1.23** *(This is an estimate based on current AWS pricing)*

**Cost Breakdown:**
- **Amazon SageMaker Notebook Instance (ml.t3.medium)**: $1.20 (24 hours × $0.05/hour)
- **KMS Customer Managed Key**: $0.033 (daily portion of $1.00/month)
- **Amazon S3 Storage (1GB file + logs)**: $0.0009 (~$0.023/GB/month)
- **KMS API Requests**: $0.0003 (minimal encryption/decryption requests)
- **Data Transfer**: $0.00 (first 100GB/month free from SRA)

### Expected Monthly Costs (US East 1):
- **Amazon SageMaker Notebook Instance (ml.t3.medium)**: ~$33.41/month (if running 24/7)
- **Amazon S3 Storage**: ~$0.023/GB/month for Standard storage
- **KMS Customer Managed Key**: $1.00/month
- **Data Transfer**: Minimal costs for downloading SRA data (first 100GB/month free)

### Cost Optimization Tips:
- **Stop the notebook instance** when not in use to avoid charges
- **Use lifecycle configurations** to automatically stop instances after periods of inactivity
- **Monitor S3 usage** and delete unnecessary files
- **Delete the CloudFormation stack** when finished to eliminate KMS key costs
- **Consider using Spot instances** for non-critical workloads

**For a typical user session (4-6 hours): ~$0.20-$0.30**

### Cleanup:
To avoid ongoing charges, delete the AWS CloudFormation stack when finished:
```bash
aws cloudformation delete-stack --stack-name opendata-sra-notebook-stack
```

## Features

### Quality Analysis Notebook

**Comprehensive Quality Control:**
- 7 FastQC-style quality metrics with scientific basis
- Per-base quality scores (Phred Q20/Q30 thresholds)
- GC content distribution analysis
- Sequence duplication detection
- Overrepresented sequence identification
- Pass/Warn/Fail indicators for all metrics
- Detailed quality summary reports

**AWS-Native Workflow:**
- Direct download from AWS Open Data Registry (S3)
- Metadata search using pysradb and NCBI SRA Run Selector
- SRA to FASTQ conversion using SRA Toolkit
- Results stored in your S3 bucket
- Optimized for cost-effective ml.t3.medium instances

**Educational Focus:**
- Step-by-step instructions for dataset discovery
- Clear explanations of quality metrics
- Scientific citations (FastQC, Phred scores)
- Instance size and cost guidance
- Troubleshooting tips

**Performance:**
- Small default dataset (7MB) for fast testing
- ~10 minute total runtime
- Disk space warnings (SRA expands 10-15x to FASTQ)
- Optimized conversion settings

## Prerequisites

- AWS account with appropriate permissions (see IAM Requirements below)
- AWS CLI configured with credentials
- Basic familiarity with AWS CloudFormation and Jupyter notebooks

### IAM Requirements

Your AWS user or role must have the following permissions to deploy this AWS CloudFormation template:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreatePolicy",
                "iam:CreateRole",
                "iam:AttachRolePolicy",
                "iam:GetRole",
                "iam:GetPolicy",
                "iam:DeleteRole",
                "iam:DeletePolicy",
                "iam:DetachRolePolicy"
            ],
            "Resource": [
                "arn:aws:iam::*:role/opendata-sra-notebook-stack*",
                "arn:aws:iam::*:policy/opendata-sra-notebook-stack*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "arn:aws:iam::*:role/opendata-sra-notebook-stack*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "sagemaker:CreateNotebookInstance",
                "sagemaker:DescribeNotebookInstance",
                "sagemaker:DeleteNotebookInstance"
            ],
            "Resource": "arn:aws:sagemaker:*:*:notebook-instance/opendata-sra-notebook*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket",
                "s3:DeleteBucket",
                "s3:PutBucketPolicy",
                "s3:PutBucketVersioning",
                "s3:PutBucketLogging",
                "s3:PutBucketPublicAccessBlock",
                "s3:PutEncryptionConfiguration"
            ],
            "Resource": "arn:aws:s3:::*-opendata-sra-bucket*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:CreateKey",
                "kms:DescribeKey",
                "kms:EnableKeyRotation",
                "kms:PutKeyPolicy",
                "kms:ScheduleKeyDeletion"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "cloudformation:*",
            "Resource": "arn:aws:cloudformation:*:*:stack/opendata-sra-notebook-stack*"
        }
    ]
}
```

**Note:** KMS requires wildcard resource permissions as key ARNs are only known after creation. If you encounter permission errors during deployment, contact your AWS administrator to attach these permissions to your user or role.

## Infrastructure Created

The AWS CloudFormation template creates:
- **Amazon SageMaker Notebook Instance** (ml.t3.medium) with Jupyter environment
- **Amazon S3 Bucket** (named `{AccountId}-opendata-sra-bucket`) for data storage
- **IAM Execution Role** with necessary permissions
- **Code Repository** connection to AWS Labs genomics examples

## Troubleshooting

### Common Issues:

#### AWS CloudFormation Stack Creation Fails
- **Issue**: Stack creation fails with permissions error
- **Solution**: Ensure your AWS credentials have permissions to create IAM roles, Amazon SageMaker resources, and Amazon S3 buckets

#### Cannot Access Notebook Instance
- **Issue**: Notebook instance shows "Pending" or "InService" but can't access Jupyter
- **Solution**: Wait 5-10 minutes for full initialization. Check the instance status in Amazon SageMaker console.

#### Amazon S3 Access Denied Errors
- **Issue**: Cannot read from or write to Amazon S3 buckets in the notebook
- **Solution**: Verify the IAM role has the correct permissions and the bucket names match the expected format

#### Notebook Kernel Issues
- **Issue**: Jupyter kernel fails to start or crashes
- **Solution**: 
  - Restart the kernel from Jupyter interface
  - Check CloudWatch logs for the notebook instance
  - Verify the lifecycle configuration completed successfully

#### High Costs
- **Issue**: Unexpected AWS charges
- **Solution**: 
  - Stop the Amazon SageMaker notebook instance when not in use
  - Monitor Amazon S3 storage usage and clean up unnecessary files
  - Set up billing alerts in AWS

### Getting Help:
- Check AWS CloudFormation events for detailed error messages
- Review Amazon SageMaker notebook instance logs in CloudWatch
- Consult AWS documentation for Amazon SageMaker and Amazon S3 troubleshooting

## Dataset

This notebook works with publicly available genomics data from the Sequence Read Archive, making it ideal for learning without requiring sensitive or proprietary datasets.

## Security

### Security Design Philosophy

This project implements AWS security best practices optimized for **educational genomics data processing**. Security controls balance learning accessibility with data protection, using public datasets that allow for educational-appropriate security postures while maintaining encryption and access controls.

**Educational Environment Priorities:**
1. **Accessibility**: Easy deployment and use for learning
2. **Cost Optimization**: Minimize infrastructure costs for students/researchers  
3. **Functional Simplicity**: Avoid complex networking that impedes data access
4. **Public Data Appropriate**: Security controls match non-sensitive genomics data

**Customer Responsibility**: While AWS provides secure infrastructure and services, customers are responsible for configuring these services appropriately for their specific security requirements and use cases.

### **✅ Implemented Security Controls**
- **Amazon S3 Encryption**: AES256 server-side encryption on all buckets
- **Amazon S3 SSL Enforcement**: Bucket policies requiring HTTPS for all requests
- **Amazon S3 Access Logging**: Detailed logging to dedicated bucket
- **Amazon S3 Public Access Blocks**: All public access prevented
- **KMS Encryption**: Customer-managed keys with automatic rotation for Amazon SageMaker
- **IAM Least Privilege**: Managed policies with specific resource ARNs
- **Instance Metadata v2**: IMDSv2 enforced on Amazon SageMaker notebooks
- **Amazon S3 Versioning**: Enabled for data protection and recovery

### **Educational vs Production Trade-offs**

**Network Access**: Amazon SageMaker notebooks have direct internet access to download genomics data from NCBI's Sequence Read Archive. Production environments should use VPC isolation with NAT Gateway (~$45/month additional cost) and VPC endpoints.

**Data Governance**: Standard Amazon S3 versioning without Object Lock enables easy cleanup for learning environments. Production workloads should evaluate Amazon S3 Object Lock for compliance requirements, though this prevents stack deletion.

**Monitoring**: Basic CloudWatch logging is sufficient for educational use. Production environments should implement AWS CloudTrail, GuardDuty, and Security Hub for detailed monitoring (~$50-200/month additional cost).

**Risk Context**: This configuration is appropriate for public genomics datasets with no PII or proprietary information. The security model prioritizes learning accessibility and cost optimization over enhanced security hardening.

### **Production Considerations**
- **Network Isolation**: Deploy Amazon SageMaker notebooks in VPC with private subnets and VPC endpoints
- **Data Governance**: Evaluate Amazon S3 Object Lock for compliance requirements (legal hold, WORM)
- **Disaster Recovery**: Implement cross-region replication for business-critical data
- **Advanced Monitoring**: Use AWS CloudTrail, GuardDuty, and Security Hub for detailed security monitoring
- **Access Management**: Implement AWS SSO and fine-grained IAM policies for production teams

### **Learn More:**
- [AWS Well-Architected Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [Amazon SageMaker Security Best Practices](https://docs.aws.amazon.com/sagemaker/latest/dg/security.html)
- [Amazon S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)
- [AWS Security Documentation](https://docs.aws.amazon.com/security/)
- [AWS Compliance Center](https://aws.amazon.com/compliance/)

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## Contributing

See [CONTRIBUTING](CONTRIBUTING.md) for more information.

## License

This library is licensed under the MIT-0 License. See the [LICENSE](LICENSE) file.

## Code of Conduct

This project has adopted the [Amazon Open Source Code of Conduct](https://aws.github.io/code-of-conduct).
For more information see the [Code of Conduct FAQ](https://aws.github.io/code-of-conduct-faq) or contact
opensource-codeofconduct@amazon.com with any additional questions or comments.
