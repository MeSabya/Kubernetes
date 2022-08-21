## Storing the Environment Variables#
The chapter is almost finished, and we do not need the cluster anymore. We want to destroy it as soon as possible. There’s no good reason to keep it running when we’re not using it. But, before we proceed with the destructive actions, we’ll create a file that will hold all the environment variables we used in this chapter. That will help us the next time we want to recreate the cluster.

```shell

echo "export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
export AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION
export ZONES=$ZONES
export NAME=$NAME
export KOPS_STATE_STORE=$KOPS_STATE_STORE" \
    >kops
```

We echoed the variables with the values into the kops file.

## Destroying Everything#
Now we can delete the cluster.

```shell
kops delete cluster \
    --name $NAME \
    --yes
```

The output is as follows.

```shell
...
Deleted kubectl config for devops23.k8s.local

Deleted cluster: "devops23.k8s.local"
```
Kops removed references of the cluster from our kubectl configuration and proceeded to delete all the AWS resources it created. Our cluster is no more. We can proceed and delete the S3 bucket as well.

```shell
aws s3api delete-bucket \
    --bucket $BUCKET_NAME
```

We will not remove the IAM resources (group, user, access key, and policies). It does not cost to keep them in AWS, and we’ll save ourselves from re-running the commands that create them. However, we will list the commands as a reference.

```shell
# Replace `[...]` with the administrative access key ID.
export AWS_ACCESS_KEY_ID=[...]

# Replace `[...]` with the administrative secret access key.
export AWS_SECRET_ACCESS_KEY=[...]

aws iam remove-user-from-group \
    --user-name kops \
    --group-name kops

aws iam delete-access-key \
    --user-name kops \
    --access-key-id $(\
    cat kops-creds | jq -r \
    '.AccessKey.AccessKeyId')

aws iam delete-user \
    --user-name kops

aws iam detach-group-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess \
    --group-name kops

aws iam detach-group-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess \
    --group-name kops

aws iam detach-group-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess \
    --group-name kops

aws iam detach-group-policy \
    --policy-arn arn:aws:iam::aws:policy/IAMFullAccess \
    --group-name kops
    
aws iam delete-group \
    --group-name kops
    
```




