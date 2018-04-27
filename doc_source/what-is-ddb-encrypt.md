# What is the Amazon DynamoDB Encryption Client?<a name="what-is-ddb-encrypt"></a>

The Amazon DynamoDB Encryption Client is a software library that helps you to protect your table data before you send it to [Amazon DynamoDB](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/)\. Encrypting your sensitive data in transit and at rest helps assure that your plaintext data isn’t available to any third party, including AWS\.

**Designed especially for DynamoDB applications**  
You don’t need to be a cryptography expert to use the DynamoDB Encryption Client\. The implementations include helper methods that are designed to work with your existing DynamoDB applications\. After you create and configure the required components, the DynamoDB Encryption Client transparently encrypts and signs your table items when you add them to a table and verifies and decrypts when you retrieve them\.

**Includes best\-practice encryption and signing**  
The DynamoDB Encryption Client includes secure, best\-practice implementations that encrypt the attribute values in each table item using a unique encryption key, and then sign the item to protect it against unauthorized changes, such as adding or deleting attributes, or swapping encrypted values\.

**Uses cryptographic material from any source**  
You can use the DynamoDB Encryption Client with encryption keys from any source, including your custom implementation or a cryptography service, such as [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) or [AWS CloudHSM](http://docs.aws.amazon.com/cloudhsm/latest/userguide/)\. The DynamoDB Encryption Client does not require AWS or any AWS service\.

**All supported programming language implementations are interoperable**  
The DynamoDB Encryption Client is an open source project on GitHub\. It is currently available in [Java](https://github.com/awslabs/aws-dynamodb-encryption-java) and [Python](https://github.com/awslabs/aws-dynamodb-encryption-python)\.  All supported programming language implementations of the DynamoDB Encryption Client are interoperable\. For example, you can encrypt data with the Java client and decrypt it with the Python client\. However, the DynamoDB Encryption Client is not compatible with the [AWS Encryption SDK](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/) or the [Amazon S3 Encryption Client](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html)\. You cannot encrypt with one client\-side library and decrypt with another\.

This developer guide is provides a conceptual overview of the DynamoDB Encryption Client, including an [introduction to its architecture](how-it-works.md), details about [how it protects DynamoDB table data](encrypted-and-signed.md) and how it differs from [DynamoDB server\-side encryption](client-server-side.md), guidance on [selecting critical components for your application](crypto-materials-providers.md), and examples in each [programming language](programming-languages.md) to help you get started\.

If you have questions or comments, use the feedback links on each page, file an issue in our [GitHub documentation repository](https://github.com/awsdocs/aws-dynamodb-encryption-docs), or ask a question in the AWS Crypto Tools forum\.

**Topics**
+ [Which Fields are Encrypted and Signed?](encrypted-and-signed.md)
+ [How the DynamoDB Encryption Client Works](how-it-works.md)
+ [Client\-Side or Server\-Side Encryption?](client-server-side.md)
+ [Amazon DynamoDB Encryption Client Concepts](concepts.md)