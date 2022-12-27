# Enumeration

!!! tip 
    When assessing AWS environments, it is recommended to configure separate testing profiles using `aws configure`.
    Each profile can then be selected for varied CLI commands using the `--profile` flag.

## IAM

Identity and Access Management (IAM) is a web service for securely controlling access to AWS services. With IAM, you can centrally manage users, security credentials such as access keys, and permissions that control which AWS resources users and applications can access.

[AWS CLI - IAM Documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/index.html#cli-aws-iam)

!!!note "Reading ARNs"
    `"arn:aws:iam::123456789012:user/DevAdmin"`  

    * `123456789012` - use this in place of `<account_id>` in the next commands
    * `DevAdmin` - use this in place of `<username>` in next commands

### Enumerating Users

#### Get current IAM identity
```
aws sts get-caller-identity  --profile <profile_name>
```

#### List IAM users
```
aws iam list-users --profile <profile_name>
```

### Enumerating Roles

#### List IAM roles
```
aws iam list-roles --profile <profile_name>
```
### Get IAM user information 
```
aws iam get-user --profile <profile_name>
```

### List IAM user inline policies
```
aws iam list-user-policies --user-name <username> --profile <profile_name>
```

### List IAM user attached policies
```
aws iam list-attached-user-policies --user-name <username> --profile <profile_name>
```

### Get IAM user policy information
```
aws iam get-user-policy --user-name <username> --policy-name <policy_arn> --profile <profile_name>
```

## Storage

### S3 Buckets

#### List all user owned S3 buckets
```
aws s3 ls --profile <profile_name> 
```

#### List all prefixes and objects in a S3 bucket
```
aws s3 ls s3://<bucket_name> --profile <profile_name>
```

### Elastic Block Stores

### RDS

## Compute

### Elastic Compute Cloud (EC2)

### Elastic Container Registry (ECR)

#### Fetch Container Credentials URI with SSRF
```
curl -i -X GET "http://container.target.flaws2.cloud/proxy/file:///proc/self/environ"
```

#### Get Container Credentials from ECS IMDS
```
curl -i -X GET "http://169.254.170.2/<version>/<uuid>"
```
/v2/credentials/2fdf39e7-50cf-4e11-a47a-533525342c80

### Elastic Kubernetes Service (EKS)

## Automated Enumeration

!!! warning
    For engagements requiring good Operational Security, it might be preferable to follow a manual approach. The following tools can be extremely noisy and easily caught by blue teams.

* [https://github.com/nccgroup/ScoutSuite](https://github.com/nccgroup/ScoutSuite) - Enumerate global security posture into HTML dashboard
* [https://github.com/initstring/cloud_enum](https://github.com/initstring/cloud_enum) - Enumerate public S3 Buckets and Applications
* [https://github.com/sa7mon/S3Scanner](https://github.com/sa7mon/S3Scanner) - Scan for open S3 buckets and dump the contents
* [https://github.com/Eilonh/s3crets_scanner](https://github.com/Eilonh/s3crets_scanner) - Trufflehog wrapper to search for secrets in S3 buckets

### Scan for unprotected S3 buckets using keyword
```
python3 cloud_enum.py -k <keyword> -l <outfile> -t <thread_counts>
```

### Scan/dump contents of open S3 buckets
```
s3scanner --threads <thread_count> scan/dump --buckets-file <bucket_list>
```

### Search for secrets in S3 buckets
*Credits: [Hunting After Secrets Accidentally Uploaded To Public S3 Buckets](https://medium.com/@hareleilon/hunting-after-secrets-accidentally-uploaded-to-public-s3-buckets-7e5bbbb80097)*
```
cd s3crets_scanner/ && python3 main.py -p <profile_name> -r <role_name>
```