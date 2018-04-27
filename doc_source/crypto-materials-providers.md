# How to Choose a Cryptographic Materials Provider<a name="crypto-materials-providers"></a>

One of the most important decisions you will make when designing your DynamoDB Encryption Client implementation is selecting a [cryptographic materials provider](concepts.md#concept-material-provider) \(CMP\)\. The CMP assembles and returns cryptographic material to the item encryptor\. It also determines how encryption and signing keys are generated, whether new key materials is generated for each item or reused, and the encryption and signing algorithms that are used\. 

You can build a compatible custom CMP or choose a CMP from the implementations provided in the DynamoDB Encryption Client libraries\. The choice of CMP might also depend on the [programming language](programming-languages.md) that you use\. Not all CMPs are available in all languages\. 

This topic describes the most common CMPs and offers some advice to help you choose the best one for your application\.

**Direct KMS Materials Provider**  
The Direct KMS Materials Provider protects your table items under an [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) [customer master key](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMK\) that never leaves AWS KMS unencrypted\. Your application does not have to generate or manage any cryptographic material\. Because It uses the CMK to generate unique encryption and signing keys for each item, this provider calls AWS KMS every time it encrypts or decrypts an item\.   
If you use AWS KMS and one AWS KMS call per transaction is practical for your application, this provider is a good choice\.  
For details, see [Direct KMS Materials Provider](direct-kms-provider.md)\.

**Wrapped Materials Provider \(Wrapped CMP\)**  
The Wrapped Materials Provider \(Wrapped CMP\) lets you generate and manage your wrapping and signing keys outside of the DynamoDB Encryption Client\.   
The Wrapped CMP generates a unique encryption key for each item\. Then it uses wrapping \(or unwrapping\) and signing keys that you supply\. As such, you determine how the wrapping and signing keys are generated and whether they are unique to each item or reused\. The Wrapped CMP is secure alternative to the [Direct KMS Provider](direct-kms-provider.md) for applications do not use AWS KMS and can safely manage cryptographic material\.   
For details, see [Wrapped Materials Provider](wrapped-provider.md)\.

**Most Recent Provider**  
The Most Recent Provider is a variation of a [Wrapped Materials Provider](wrapped-provider.md) that generates a new encryption key for each item, but caches and reuses wrapping and signing keys\. The reusable keys are stored in encrypted form a DynamoDB table that can support multiple hosts simultaneously\. The Most Recent Provider is good choice for high\-volume applications that are sensitive to latency and can reuse some cryptographic material\.  
You can use encryption materials from any source to protect your wrapping and signing keys, including material that you generate, or material from [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) , or [AWS CloudHSM](http://docs.aws.amazon.com/cloudhsm/latest/userguide/)\.   
For details, see [Most Recent Provider](most-recent-provider.md)\.

**Static Materials Provider**  
The Static Materials Provider is designed for testing and proof\-of\-concept demonstrations, not production use\. It does not generate any unique cryptographic material for each item\. It returns the same encryption and signing keys that you supply, and those keys are used directly to encrypt, decrypt, and sign your table items\.   
The [Asymmetric Static Provider](https://github.com/awslabs/aws-dynamodb-encryption-java/blob/master/src/main/java/com/amazonaws/services/dynamodbv2/datamodeling/encryption/providers/AsymmetricStaticProvider.java) in the Java library is not a static provider\. It calls the [Wrapped CMP](wrapped-provider.md), which generates unique encryption material for each item\. It is safe for production use, but you should use the Wrapped CMP directly whenever possible\.

**Topics**
+ [Direct KMS Materials Provider](direct-kms-provider.md)
+ [Wrapped Materials Provider](wrapped-provider.md)
+ [Most Recent Provider](most-recent-provider.md)
+ [Static Materials Provider](static-provider.md)