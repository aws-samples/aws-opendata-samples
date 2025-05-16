
# Migrating Data to Amazon S3 with Rclone

## Overview

This guidance provides prescriptive steps to efficiently perform a server-side copy from a source Amazon S3 bucket to a destination Amazon S3 bucket using Rclone. Rclone is a powerful, open-source command-line tool inspired by `rsync`, capable of transferring data between various cloud storage providers, including Amazon S3.

This pattern is ideal for customers seeking scalable, high-performance data migration with parallelism, operational simplicity, and minimal cost.

> **Important Notice for Open Data Sponsorship Program Data Providers:**  
> If you are a data provider within the AWS Open Data Sponsorship Program, you must execute this guidance in your paid AWS account. Do **not** perform these steps in your sponsored Open Data program AWS account, as launching AWS resources beyond what is necessary to host your dataset is restricted by the program's terms and conditions.

---

## What You'll Do

You will:

- Launch an Amazon EC2 instance
- Create and attach an IAM role with permissions to read from your source S3 bucket and write to your destination S3 bucket
- Install and configure Rclone on your EC2 instance
- Execute a server-side copy operation
- Verify successful migration

---

## Prerequisites

- An active AWS account
- A destination Amazon S3 bucket in the same AWS Region as the source bucket. If you don’t have a destination bucket, follow the instructions in [Creating a bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html). The bucket can be public or private, but you must have permission to read from and write to it to complete this guidance.

---

## Recommended Best Practices

- **As of May 13, 2025**, Rclone requires both the source and destination S3 buckets to be in the **same AWS Region** and referenced by the **same remote configuration** to support server-side copy. This allows Amazon S3 to perform internal object transfers without routing through your EC2 instance. Server-side copy is faster and more efficient for copying objects than client-side copy as you don't need the client to download the object, then re-upload it to the destination.
- Use EC2 instances with high network performance (e.g., c7g.large or larger Graviton-based instances) to maximize transfer speed and minimize cost.

---

## Setup and Configuration

### Step 1: Launch an EC2 Instance

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/) and navigate to the EC2 service.
   
2. [Launch an Amazon EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/LaunchingAndUsingInstances.html) in the same region as your S3 buckets.  

---

### Step 2: Create and Attach an IAM Role

1. [Create an IAM role for EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html#roles-creatingrole-service-console).
   
2. Create a [customer managed policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html) that grants access to the source and destination S3 buckets, and attach it to your IAM role. Replace `<source-bucket>` and `<destination-bucket>` with your actual bucket names:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": [
        "arn:aws:s3:::<destination-bucket>",
        "arn:aws:s3:::<source-bucket>"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<source-bucket>/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::<destination-bucket>/*"
    }
  ]
}
```

---

### Step 3: Install Rclone

1. Connect to your EC2 instance using SSH or Session Manager: [Connect to your EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect.html). Ensure that only authorized users have access to your instance.

2. Download and install Rclone from the official site: https://rclone.org/downloads/

> **Verify Rclone Signature:**  
> To ensure the integrity of your download, follow the [release signing instructions](https://rclone.org/release_signing/) provided by Rclone.

---

### Step 4: Configure Rclone

Rclone must use the **same remote configuration** for both source and destination buckets to enable server-side copy.

1. Create the config file:
```bash
vi ~/.config/rclone/rclone.conf
```

2. Add the following (replace `<buckets-region>` with your actual region, e.g., `us-east-1`):

```ini
[s3]
type = s3
provider = AWS
region = <buckets-region>
env_auth = true
acl = private
```

---

### Step 5: Verify Configuration

Ensure Rclone can list your S3 buckets:

```bash
rclone listremotes                 # Should print: s3:
rclone lsd s3:<source-bucket>      # Lists top-level prefixes in the source bucket
rclone lsd s3:<destination-bucket> # Will be empty if the destination bucket is new
```

---

## Migrate Data Using Rclone

### Step 1: Execute Data Transfer

Run the Rclone copy command to initiate the server-side copy. The example below includes performance-optimized options, which you can modify based on your use case. For a full list of available flags and guidance on tuning, see the [Rclone documentation](https://rclone.org/docs/).

```bash
rclone copy -P   --transfers=256   --checkers=256   --s3-upload-concurrency=64   --multi-thread-streams=64   --s3-chunk-size=64M   --s3-force-path-style=true   --log-file=transfer-log.log   s3:<source-bucket> s3:<destination-bucket>
```

#### Explanation of Command Options

- `-P`: Show progress and performance stats
- `--transfers`: Number of parallel file transfers
- `--checkers`: Number of file checks in parallel
- `--s3-upload-concurrency`: Concurrent uploads per file
- `--multi-thread-streams`: Multi-threaded uploads/downloads
- `--s3-chunk-size`: Chunk size for multipart upload
- `--s3-force-path-style`: Use path-style S3 addressing
- `--log-file`: Logs all operations for review

---

### Optional Performance Enhancements

Use the following flags for faster execution, with reduced validation:

```bash
--size-only --s3-no-check-bucket --no-check-dest
```

- `--size-only`: Skip checks on timestamps/checksums
- `--s3-no-check-bucket`: Skip bucket existence check
- `--no-check-dest`: Skip checking destination before writing

---

### Step 2: Verify Transfer Success

To confirm the data was successfully copied, list the objects in your destination bucket. This command may take time to complete depending on the number of objects copied:

```bash
rclone ls s3:<destination-bucket>
```

Compare object counts between the source and destination buckets:

```bash
# Source bucket (for public buckets add --no-sign-request)
aws s3 ls --no-sign-request --recursive s3://<source-bucket>/ | wc -l

# Destination bucket
aws s3 ls --recursive s3://<destination-bucket>/ | wc -l
```

---

## Additional Resources

- [Amazon S3 User Guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [IAM Roles for EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html)
- [Rclone Documentation](https://rclone.org/docs/)

---

By following this guidance, you can efficiently migrate data between Amazon S3 buckets using Rclone, leveraging the scalability and performance of AWS infrastructure.