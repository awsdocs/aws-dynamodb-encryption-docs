# What is the Amazon DynamoDB Encryption Client?<a name="what-is-ddb-encrypt"></a>

The Amazon DynamoDB Encryption Client is a software library that enables you to include client\-side encryption in your [Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/) design\. The DynamoDB Encryption Client is designed to be implemented in new, unpopulated databases\. Encrypting your sensitive data in transit and at rest helps ensure that your plaintext data isn’t available to any third party, including AWS\. The DynamoDB Encryption Client is provided free of charge under the Apache 2\.0 license\.

**Important**  
The DynamoDB Encryption Client does not support the encryption of existing, unencrypted DynamoDB table data\.

This developer guide provides a conceptual overview of the DynamoDB Encryption Client, including an [introduction to its architecture](how-it-works.md), details about [how it protects DynamoDB table data](encrypted-and-signed.md) and how it differs from [DynamoDB server\-side encryption](client-server-side.md), guidance on [selecting critical components for your application](crypto-materials-providers.md), and examples in each [programming language](programming-languages.md) to help you get started\.

The DynamoDB Encryption Client has the following benefits:

**Designed especially for DynamoDB applications**  
You don’t need to be a cryptography expert to use the DynamoDB Encryption Client\. The implementations include helper methods that are designed to work with your existing DynamoDB applications\.   
After you create and configure the required components, the DynamoDB Encryption Client transparently encrypts and signs your table items when you add them to a table, and verifies and decrypts them when you retrieve them\.  
The DynamoDB Encryption Client supports most Amazon DynamoDB features, including [global tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)\. However, you might need to make some configuration changes if you're using an older version of global tables\. For details, see [Issues with older version global tables](troubleshooting.md#fix-global-tables)\.\.

**Includes secure encryption and signing**  
The DynamoDB Encryption Client includes secure implementations that encrypt the attribute values in each table item using a unique encryption key, and then sign the item to protect it against unauthorized changes, such as adding or deleting attributes, or swapping encrypted values\.

**Uses cryptographic materials from any source**  
You can use the DynamoDB Encryption Client with encryption keys from any source, including your custom implementation or a cryptography service, such as [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) or [AWS CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/)\. The DynamoDB Encryption Client doesn't require an AWS account or any AWS service\.

**Programming language implementations are interoperable**  
The DynamoDB Encryption Client libraries are developed in open source projects on GitHub\. They are currently available in [Java](https://github.com/aws/aws-dynamodb-encryption-java/) and [Python](https://github.com/aws/aws-dynamodb-encryption-python/)\.  All supported programming language implementations of the DynamoDB Encryption Client are interoperable\. For example, you can encrypt data with the Java client and decrypt it with the Python client\.   
However, the DynamoDB Encryption Client is not compatible with the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/) or the [Amazon S3 Encryption Client](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html)\. You cannot encrypt with one client\-side library and decrypt with another\.

## Developed in open\-source repositories<a name="esdk-repos"></a>

The Amazon DynamoDB Encryption Client is developed in open\-source repositories on GitHub\. You can use these repositories to view the code, read and submit issues, and find information that is specific to your language implementation\.
+ DynamoDB Encryption Client for Java — [aws\-dynamodb\-encryption\-java](https://github.com/aws/aws-dynamodb-encryption-java/)
+ DynamoDB Encryption Client for Python — [aws\-dynamodb\-encryption\-python](https://github.com/aws/aws-dynamodb-encryption-python/)

## Support and maintenance<a name="support"></a>

The DynamoDB Encryption Client uses the same [maintenance policy](https://docs.aws.amazon.com/sdkref/latest/guide/maint-policy.html) that the AWS SDK and Tools use, including its versioning and life\-cycle phases\. As a best practice, we recommend that you use the latest available version of the DynamoDB Encryption Client for your programming language, and upgrade as new versions are released\.

Each programming language implementation of the DynamoDB Encryption Client is developed in a separate open\-source GitHub repository\. The life\-cycle and support phase of each version is likely to vary among repositories\. For example, a given version of the DynamoDB Encryption Client might be in the general availability \(full support\) phase in one programming language, but the end\-of\-support phase in a different programming language\. We recommend that you use a fully supported version whenever possible and avoid versions that are no longer supported\.

To find the life\-cycle phase of DynamoDB Encryption Client versions for your programming language, see the `SUPPORT_POLICY.rst` file in each DynamoDB Encryption Client repository\.
+ DynamoDB Encryption Client for Java — [SUPPORT\_POLICY\.rst](https://github.com/aws/aws-dynamodb-encryption-java/blob/master/SUPPORT_POLICY.rst)
+ DynamoDB Encryption Client for Python — [SUPPORT\_POLICY\.rst](https://github.com/aws/aws-dynamodb-encryption-python/blob/master/SUPPORT_POLICY.rst)

For more information, see the [AWS SDKs and Tools maintenance policy](https://docs.aws.amazon.com/sdkref/latest/guide/maint-policy.html) in the AWS SDKs and Tools Reference Guide\.

## Sending feedback<a name="feedback"></a>

We welcome your feedback\! If you have a question or comment, or an issue to report, please use the following resources\.
+ If you discover a potential security vulnerability in the DynamoDB Encryption Client, please [notify AWS security](https://aws.amazon.com/security/vulnerability-reporting/)\. Do not create a public GitHub issue\.
+ To provide feedback on the DynamoDB Encryption Client, file an issue in the [aws\-dynamodb\-encryption\-java](https://github.com/aws/aws-dynamodb-encryption-java/) or [aws\-dynamodb\-encryption\-python](https://github.com/aws/aws-dynamodb-encryption-python/) GitHub repository\.
+ To provide feedback on this documentation, use the feedback link on any page\. You can also file an issue or contribute to [aws\-dynamodb\-encryption\-docs](https://github.com/awsdocs/aws-dynamodb-encryption-docs/), the open\-source repository for this documentation on GitHub\.