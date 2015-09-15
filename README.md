# EC2 Keypair Generator

*Note:* Thought of, implemented and tested whilst drunk at 1am. Working, but use at your own risk for now!

| Staging | Production |
|:-:|:-:|
|[![Build Status](http://drone.stocktio.com/api/badge/github.com/Stockflare/lambda-ec2-key-generator/status.svg?branch=master)](http://drone.stocktio.com/github.com/Stockflare/lambda-ec2-key-generator)| --- |

If you're lazy like us at Stockflare; creating, uploading, managing, deleting and destroying EC2 Keypairs is tasking (I know I need a coffee afterwards). This function automates the task of creating, updating and deleting EC2 Keypairs.

It will place all private keys into a designated S3 bucket. Naturally, this bucket needs to be locked down and at the [very least encrypted](http://aws.amazon.com/documentation/kms/).

We all share this bucket/directory as a shared mount which makes it painless for us all to access and keep keys up-to-date.

**Note:** The parameter `StackOutputsArn` has been declared elsewhere in the template. Or in our case, we use the [launcher gem](http://github.com/Stockflare/launcher) to automatically discover it.

```
"KeyGenerator": {
  "Type": "Custom::StackOutputs",
  "Properties": {
    "ServiceToken": { "Ref" : "StackOutputsArn" },
    "StackName" : "key-generator"
  }
},

"KeyPair": {
  "Type": "Custom::KeyPairGenerator",
  "Properties": {
    "ServiceToken": { "Fn::GetAtt" : ["KeyGenerator", "KeyPairArn"] },
    "Region" : { "Ref" : "AWS::Region" },
    "KeyName" : { "Ref" : "AWS::StackName" },
    "Bucket" : { "Ref" : "PrivateS3KeyBucket" }
  }
}
```

Easily retrieve your new EC2 Keypair name elsewhere in your template like:

```
{ "Fn::GetAtt": [ "KeyPair", "Name" ] }
```

Note that whilst this is passed back as an attribute, for now it is equivalent to the property passed to `KeyName` inside the resource definition.

### Mounting the directory from S3

You can use a tool like [s3fs](https://github.com/s3fs-fuse/s3fs-fuse). It makes mounting directories very easy, once installed you can use the following command to mount the S3 Keys Bucket inside your machine. Making key management amongst lots of developers very easy:

```
s3fs mycompany.com-keys:/ec2 ~/.s3/MyCompany/sub-dir/keys -o passwd_file=~/.s3/MyCompany/credentials
```
