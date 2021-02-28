---
title: 'Upload image to S3 using AWS CLI'
date: 2021-02-28
permalink: /posts/2021/02/upload-image-to-s3-using-aws-cli/
tags:
- aws
- s3
---

## Prerequisites
Unix-based OS.

## Install

```shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

[Click here for detail.](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)

## Configure

Configure AWS CLI using following command
```shell
aws configure
```
This initiates an interactive configuration process that prompts us for our settings
<pre>
AWS Access Key ID [None]: <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-creds">Get access key guide.</a>
AWS Secret Access Key [None]: <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-creds">Get access key guide.</a>
Default region name [None]: ap-southeast-1 (<a href="https://stackoverflow.com/questions/61091762/what-is-the-quickest-way-to-find-the-default-region-for-a-new-aws-account">You can type any valid region name</a>)
Default output format [None]: json
</pre>

## Copy file to bucket

Get list AWS S3 bucket
```shell
aws s3 ls
```

Get list folder in your AWS S3 bucket
```shell
aws s3 ls s3://<S3_BUCKET_NAME> --recursive --human-readable --summarize | awk '{print $5}' | awk -F '/' '/\// {print $1}' | sort -u
```

Copy file from local to bucket

```shell
aws s3 cp <path-to-file-from-local> s3://<S3_BUCKET_NAME>/<folder-name> --acl public-read
```

The above command don't return object's url for you but you can construct object's url according to `path style` access as follows

```shell
https://<region>.amazonaws.com/<bucket-name>/<key>

Where <region> is something like s3-ap-southeast-2
```

[Note that path style url will be replaced by virtual hosted‐style and deprecated soon ](https://aws.amazon.com/vi/blogs/aws/amazon-s3-path-deprecation-plan-the-rest-of-the-story/). [See more here](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RESTAPI.html).


-------------
Opinion are my own. Thank you for reading.