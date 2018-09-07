# What Is the Amazon DynamoDB Encryption Client?<a name="what-is-ddb-encrypt"></a>

The Amazon DynamoDB Encryption Client is a software library that helps you to protect your table data before you send it to [Amazon DynamoDB](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/)\. Encrypting your sensitive data in transit and at rest helps ensure that your plaintext data isn’t available to any third party, including AWS\.

This developer guide provides a conceptual overview of the DynamoDB Encryption Client, including an [introduction to its architecture](how-it-works.md), details about [how it protects DynamoDB table data](encrypted-and-signed.md) and how it differs from [DynamoDB server\-side encryption](client-server-side.md), guidance on [selecting critical components for your application](crypto-materials-providers.md), and examples in each [programming language](programming-languages.md) to help you get started\.

The DynamoDB Encryption Client has the following benefits:

**Designed Especially for DynamoDB Applications**  
You don’t need to be a cryptography expert to use the DynamoDB Encryption Client\. The implementations include helper methods that are designed to work with your existing DynamoDB applications\.   
After you create and configure the required components, the DynamoDB Encryption Client transparently encrypts and signs your table items when you add them to a table, and verifies and decrypts them when you retrieve them\.

**Includes Secure Encryption and Signing**  
The DynamoDB Encryption Client includes secure implementations that encrypt the attribute values in each table item using a unique encryption key, and then sign the item to protect it against unauthorized changes, such as adding or deleting attributes, or swapping encrypted values\.

**Uses Cryptographic Materials from Any Source**  
You can use the DynamoDB Encryption Client with encryption keys from any source, including your custom implementation or a cryptography service, such as [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) or [AWS CloudHSM](http://docs.aws.amazon.com/cloudhsm/latest/userguide/)\. The DynamoDB Encryption Client doesn't require an AWS account or any AWS service\.

**All Supported Programming Language Implementations Are Interoperable**  
The DynamoDB Encryption Client libraries are developed in open source projects on GitHub\. They are currently available in [Java](https://github.com/aws/aws-dynamodb-encryption-java/) and [Python](https://github.com/aws/aws-dynamodb-encryption-python/)\.  All supported programming language implementations of the DynamoDB Encryption Client are interoperable\. For example, you can encrypt data with the Java client and decrypt it with the Python client\.   
However, the DynamoDB Encryption Client is not compatible with the [AWS Encryption SDK](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/) or the [Amazon S3 Encryption Client](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html)\. You cannot encrypt with one client\-side library and decrypt with another\.

If you have questions about using the DynamoDB Encryption Client, read and post on the [AWS Crypto Tools Discussion Forum](https://forums.aws.amazon.com/forum.jspa?forumID=302), file an issue in the GitHub repository for the [Java](https://github.com/aws/aws-dynamodb-encryption-java/) or [Python](https://github.com/aws/aws-dynamodb-encryption-python/) library, or contact [AWS Support](https://console.aws.amazon.com/support/home)\.

To suggest changes to any page in this guide, choose the feedback link in the lower\-right corner of the page, or the GitHub link in the upper\-right corner of the page\. You can also file an issue in the [aws\-dynamodb\-encryption\-docs](https://github.com/awsdocs/aws-dynamodb-encryption-docs) GitHub repository for this guide\.

The Amazon DynamoDB Encryption Client is provided free of charge under the Apache license\.

**Topics**
+ [Which Fields Are Encrypted and Signed?](encrypted-and-signed.md)
+ [How the DynamoDB Encryption Client Works](how-it-works.md)
+ [Client\-Side or Server\-Side Encryption?](client-server-side.md)
+ [Amazon DynamoDB Encryption Client Concepts](concepts.md)