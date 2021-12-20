# Troubleshooting issues in your DynamoDB Encryption Client application<a name="troubleshooting"></a>

This section describes problems that you might encounter when using the DynamoDB Encryption Client and offers suggestions for resolving them\.

To provide feedback on the DynamoDB Encryption Client, file an issue in the [aws\-dynamodb\-encryption\-java](https://github.com/aws/aws-dynamodb-encryption-java/) or [aws\-dynamodb\-encryption\-python](https://github.com/aws/aws-dynamodb-encryption-python/) GitHub repository\.

To provide feedback on this documentation, use the feedback link on any page\. You can also file an issue or contribute to [aws\-dynamodb\-encryption\-docs](https://github.com/awsdocs/aws-dynamodb-encryption-docs/), the open\-source repository for this documentation on GitHub\.

**Topics**
+ [Access denied](#kms-permissions)
+ [Signature verification fails](#change-data-model)
+ [Issues with older version global tables](#fix-global-tables)
+ [Poor performance of the Most Recent Provider](#mrp-ttl-delay)

## Access denied<a name="kms-permissions"></a>

**Problem**: Your application is denied access to a resource that it needs\.

**Suggestion**: Learn about the required permissions and add them to the security context in which your application runs\.

**Details**

To run an application that uses the a DynamoDB Encryption Client library, the caller must have permission to use its components\. Otherwise, they will be denied access to the required elements\. 
+ The DynamoDB Encryption Client does not require an Amazon Web Services \(AWS\) account or depend on any AWS service\. However, if your application uses AWS, you need [an AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) and [users who have permission](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) to use the account\.
+ The DynamoDB Encryption Client does not require Amazon DynamoDB\. However, If the application that uses the client creates DynamoDB tables, puts items into a table, or gets items from a table, the caller must have permission to use the required DynamoDB operations in your AWS account\. For details, see the [access control topics](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/access-control-overview.html) in the *Amazon DynamoDB Developer Guide*\.
+ If your application uses a [client helper class](python-using.md#python-helpers) in the DynamoDB Encryption Client for Python, the caller must have permission to call the DynamoDB [DescribeTable](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_DescribeTable.html) operation\.
+ The DynamoDB Encryption Client does not require AWS Key Management Service\(AWS KMS\)\. However, if your application uses a [Direct KMS Materials Provider](direct-kms-provider.md), or it uses a [Most Recent Provider](most-recent-provider.md) with a provider store that uses AWS KMS, the caller must have permission to use the AWS KMS [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations\.

## Signature verification fails<a name="change-data-model"></a>

**Problem**: An item cannot be decrypted because signature verification fails\. The item also might not be encrypted and signed as you intend\.

**Suggestion**: Be sure that the attribute actions that you provide account for all attributes in the item\. When decrypting an item, be sure to provide attribute actions that match the actions used to encrypt the item\.

**Details**

The [attribute actions](concepts.md#attribute-actions) that you provide tell the DynamoDB Encryption Client which attributes to encrypt and sign, which attributes to sign \(but not encrypt\), and which to ignore\. 

If the attribute actions that you specify do not account for all attributes in the item, the item might not be encrypted and signed the way that you intend\. More importantly, if your attribute actions do not account for all attributes in the item, the item might not be encrypted and signed the way that you intend\. 

If the attribute actions that you provide when decrypting an item differ from the attribute actions that you provided when encrypting the item, the signature verification might fail\. This is a particular problem for distributed applications in which new attribute actions might not have propagated to all hosts\.

Signature validation errors are difficult to resolve\. For help preventing them, take extra precautions when changing your data model\. For details, see [Changing your data model](data-model.md)\.

## Issues with older version global tables<a name="fix-global-tables"></a>

**Problem**: Items in an older version Amazon DynamoDB global table cannot be decrypted because signature verification fails\.

**Suggestion**: Set attribute actions so the reserved replication fields are not encrypted or signed\.

**Details**

You can use the DynamoDB Encryption Client with [DynamoDB global tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)\. We recommend that you use global tables with a [multi\-Region KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html) and replicate the KMS key into all AWS Regions where the global table is replicated\.

Beginning with global tables [version 2019\.11\.21](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/globaltables.V2.html), you can use global tables with the DynamoDB Encryption Client without any special configuration\. However, if you use global tables [version 2017\.11\.29](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/globaltables.V1.html), you must ensure that reserved replication fields are not encrypted or signed\.

If you are using the global tables version 2017\.11\.29, you must set the attribute actions for the following attributes to `DO_NOTHING` in [Java](java-using.md#attribute-actions-java) or `@DoNotTouch` in [Python](python-using.md#python-attribute-actions)\.
+ `aws:rep:deleting`
+ `aws:rep:updatetime`
+ `aws:rep:updateregion`

If you are using any other version of global tables, no action is required\.

## Poor performance of the Most Recent Provider<a name="mrp-ttl-delay"></a>

**Problem**: Your application is less responsive, especially after updating to a newer version of the DynamoDB Encryption Client\.

**Suggestion**: Adjust the time\-to\-live value and cache size\.

**Details**

The Most Recent Provider is designed to improve the performance of applications that use the DynamoDB Encryption Client by allowing limited reuse of cryptographic materials\. When you configure the Most Recent Provider for your application, you have to balance improved performance with the security concerns that arise from caching and reuse\. 

In newer versions of the DynamoDB Encryption Client, the time\-to\-live \(TTL\) value determines how long cached cryptographic material providers \(CMPs\) can be used\. The TTL also determines how often the Most Recent Provider checks for a new version of the CMP\. 

If your TTL is too long, your application might violate your business rules or security standards\. If your TTL is too brief, frequent calls to the provider store can cause your provider store to throttle requests from your application and other applications that share your service account\. To resolve this issue, adjust the TTL and cache size to a value that meets your latency and availability goals and conforms to your security standards\. For details, see [Setting a time\-to\-live value](most-recent-provider.md#most-recent-provider-ttl)\.