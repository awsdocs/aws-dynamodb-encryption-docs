# Direct KMS Materials Provider<a name="direct-kms-provider"></a>

The *Direct KMS Materials Provider* \(Direct KMS Provider\) protects your table items under an [AWS KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) that never leaves [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) unencrypted\. This [cryptographic materials provider](concepts.md#concept-material-provider) returns a unique encryption key and signing key for every table item\. To do so, it calls AWS KMS every time you encrypt or decrypt an item\.

If you're processing DynamoDB items at a high frequency and large scale, you might exceed the AWS KMS [requests\-per\-second limits](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html#requests-per-second), causing processing delays\. If you need to exceed a limit, create a case in the [AWS Support Center](https://console.aws.amazon.com/support/home)\. You might also consider using a cryptographic materials provider with limited key reuse, such as the [Most Recent Provider](most-recent-provider.md)\.

To use the Direct KMS Provider, the caller must have [an AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), at least one AWS KMS key, and permission to call the [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations on the AWS KMS key\. The AWS KMS key must be a symmetric encryption key; the DynamoDB Encryption Client does not support asymmetric encryption\. If you are using a [DynamoDB global table](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html), you might want to specify an [AWS KMS multi\-Region key](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html)\. For details, see [How to use it](#provider-kms-how-to-use)\.

**Note**  
When you use the Direct KMS Provider, the names and values of your primary key attributes appear in plaintext in the [AWS KMS encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context) and AWS CloudTrail logs of related AWS KMS operations\. However, the DynamoDB Encryption Client never exposes the plaintext of any encrypted attribute values\.

The Direct KMS Provider is one of several [cryptographic materials providers](concepts.md#concept-material-provider) \(CMPs\) that the DynamoDB Encryption Client supports\. For information about the other CMPs, see [How to choose a cryptographic materials provider](crypto-materials-providers.md)\.

**For example code, see:**
+ Java: [AwsKmsEncryptedItem](https://github.com/aws/aws-dynamodb-encryption-java/blob/master/examples/src/main/java/com/amazonaws/examples/AwsKmsEncryptedItem.java)
+ Python: [aws\-kms\-encrypted\-table](https://github.com/aws/aws-dynamodb-encryption-python/blob/master/examples/src/dynamodb_encryption_sdk_examples/aws_kms_encrypted_table.py), [aws\-kms\-encrypted\-item](https://github.com/aws/aws-dynamodb-encryption-python/blob/master/examples/src/dynamodb_encryption_sdk_examples/aws_kms_encrypted_item.py)

**Topics**
+ [How to use it](#provider-kms-how-to-use)
+ [How it works](#provider-kms-how-it-works)

## How to use it<a name="provider-kms-how-to-use"></a>

To create a Direct KMS Provider, use the key ID parameter to specify a symmetric encryption [KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) in your account\. The value of the key ID parameter can be the key ID, key ARN, alias name, or alias ARN of the AWS KMS key\. For details about the key identifiers, see [Key identifiers](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id) in the *AWS Key Management Service Developer Guide*\.

The Direct KMS Provider requires a symmetric encryption KMS key\. You cannot use an asymmetric KMS key\. However, you can use a multi\-Region KMS key, a KMS key with imported key material, or a KMS key in a custom key store\. You must have [kms:GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) permission on the KMS key\. As such, you must use a customer managed key, not an AWS managed or AWS owned KMS key\.

The DynamoDB Encryption Client for Python determines the Region for calling AWS KMS from the Region in the key ID parameter value, if it includes one\. Otherwise, it uses the Region in the AWS KMS client, if you specify one, or the Region that you configure in the AWS SDK for Python \(Boto3\)\. For information about Region selection in Python, see [Configuration](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/configuration.html) in the AWS SDK for Python \(Boto3\) API Reference\.

The DynamoDB Encryption Client for Java determines the Region for calling AWS KMS from the Region in the AWS KMS client, if the client you specify includes a Region\. Otherwise, it uses the Region that you configure in the AWS SDK for Java\. For information about Region selection in the AWS SDK for Java, see [AWS Region selection](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/java-dg-region-selection.html) in the AWS SDK for Java Developer Guide\.

------
#### [ Java ]

```
// Replace the example key ARN and Region with valid values for your application
final String keyArn = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
final String region = 'us-west-2'
      
final AWSKMS kms = AWSKMSClientBuilder.standard().withRegion(region).build();
final DirectKmsMaterialProvider cmp = new DirectKmsMaterialProvider(kms, keyArn);
```

------
#### [ Python ]

The following example uses the key ARN to specify the AWS KMS key\. If your key identifier doesn't include an AWS Region, the DynamoDB Encryption Client gets the Region from the configured Botocore session, if there is one, or from Boto defaults\.

```
# Replace the example key ID with a valid value
kms_key = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
kms_cmp = AwsKmsCryptographicMaterialsProvider(key_id=kms_key)
```

------

If you are using [Amazon DynamoDB global tables](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html), we recommend that you encrypt your data under an AWS KMS multi\-Region key\. Multi\-Region keys are AWS KMS keys in different AWS Regions that can be used interchangeably because they have the same key ID and key material\. For details, see [Using multi\-Region keys](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html) in the *AWS Key Management Service Developer Guide*\.

**Note**  
If you are using the global tables [version 2017\.11\.29](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/globaltables.V1.html), you must set attribute actions so the reserved replication fields are not encrypted or signed\. For details, see [Issues with older version global tables](troubleshooting.md#fix-global-tables)\.

To use a multi\-Region key with the DynamoDB Encryption Client, create a multi\-Region key and replicate it into the Regions in which your application runs\. Then configure the Direct KMS Provider to use the multi\-Region key in the Region in which the DynamoDB Encryption Client calls AWS KMS\.

The following example configures the DynamoDB Encryption Client to encrypt data in the US East \(N\. Virginia\) \(us\-east\-1\) Region and decrypt it in the US West \(Oregon\) \(us\-west\-2\) Region using a multi\-Region key\.

------
#### [ Java ]

In this example, the DynamoDB Encryption Client gets the Region for calling AWS KMS from the Region in the AWS KMS client\. The `keyArn` value identifies a multi\-Region key in the same Region\.

```
// Encrypt in us-east-1

// Replace the example key ARN and Region with valid values for your application
final String usEastKey = 'arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab'
final String region = 'us-east-1'
      
final AWSKMS kms = AWSKMSClientBuilder.standard().withRegion(region).build();
final DirectKmsMaterialProvider cmp = new DirectKmsMaterialProvider(kms, usEastKey);
```

```
// Decrypt in us-west-2

// Replace the example key ARN and Region with valid values for your application
final String usWestKey = 'arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab'
final String region = 'us-west-2'
      
final AWSKMS kms = AWSKMSClientBuilder.standard().withRegion(region).build();
final DirectKmsMaterialProvider cmp = new DirectKmsMaterialProvider(kms, usWestKey);
```

------
#### [ Python ]

In this example, the DynamoDB Encryption Client gets the Region for calling AWS KMS from the Region in the key ARN\.

```
# Encrypt in us-east-1

# Replace the example key ID with a valid value
us_east_key = 'arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab'
kms_cmp = AwsKmsCryptographicMaterialsProvider(key_id=us_east_key)
```

```
# Decrypt in us-west-2

# Replace the example key ID with a valid value
us_west_key = 'arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab'
kms_cmp = AwsKmsCryptographicMaterialsProvider(key_id=us_west_key)
```

------

## How it works<a name="provider-kms-how-it-works"></a>

The Direct KMS Provider returns encryption and signing keys that are protected by an AWS KMS key that you specify, as shown in the following diagram\.

![\[The input, processing, and output of the Direct KMS Provider in the DynamoDB Encryption Client\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/directKMS.png)
+ To generate encryption materials, the Direct KMS Provider asks AWS KMS to [generate a unique data key](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) for each item using an AWS KMS key that you specify\. It derives encryption and signing keys for the item from the plaintext copy of the [data key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys), and then returns the encryption and signing keys, along with the encrypted data key, which is stored in the [material description attribute](concepts.md#material-description) of the item\. 

  The item encryptor uses the encryption and signing keys and removes them from memory as soon as possible\. Only the encrypted copy of the data key from which they were derived is saved in the encrypted item\.
+ To generate decryption materials, the Direct KMS Provider asks AWS KMS to decrypt the encrypted data key\. Then, it derives verification and signing keys from the plaintext data key, and returns them to the item encryptor\.

  The item encryptor verifies the item and, if verification succeeds, decrypts the encrypted values\. Then, it removes the keys from memory as soon as possible\.

### Get encryption materials<a name="direct-kms-get-encryption-materials"></a>

This section describes in detail the inputs, outputs, and processing of the Direct KMS Provider when it receives a request for encryption materials from the [item encryptor](concepts.md#item-encryptor)\.

**Input ** \(from the application\)
+ The key ID of an AWS KMS key\. 

**Input** \(from the item encryptor\)
+ [DynamoDB encryption context](concepts.md#encryption-context)

**Output** \(to the item encryptor\)
+ Encryption key \(plaintext\)
+ Signing key
+ In [actual material description](concepts.md#material-description): These values are saved in the material description attribute that the client adds to the item\.
  + amzn\-ddb\-env\-key: Base64\-encoded data key encrypted by the AWS KMS key
  + amzn\-ddb\-env\-alg: Encryption algorithm, by default [AES/256](https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines/archived-crypto-projects/aes-development)
  + amzn\-ddb\-sig\-alg: Signing algorithm, by default, [HmacSHA256/256](https://en.wikipedia.org/wiki/HMAC)
  + amzn\-ddb\-wrap\-alg: kms

**Processing**

1. The Direct KMS Provider sends AWS KMS a request to use the specified AWS KMS key to [generate a unique data key](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) for the item\. The operation returns a plaintext key and a copy that is encrypted under the AWS KMS key\. This is known as the *initial key material*\.

   The request includes the following values in plaintext in [AWS KMS encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context)\. These non\-secret values are cryptographically bound to the encrypted object, so the same encryption context is required on decrypt\. You can use these values to identify the call to AWS KMS in [AWS CloudTrail logs](https://docs.aws.amazon.com/kms/latest/developerguide/monitoring-overview.html)\.
   + amzn\-ddb\-env\-alg – Encryption algorithm, by default AES/256
   + amzn\-ddb\-sig\-alg – Signing algorithm, by default HmacSHA256/256
   + \(Optional\) aws\-kms\-table – *table name*
   + \(Optional\) *partition key name* – *partition key value* \(binary values are Base64\-encoded\)
   + \(Optional\) *sort key name* – *sort key value* \(binary values are Base64\-encoded\)

   The Direct KMS Provider gets the values for the AWS KMS encryption context from the [DynamoDB encryption context](concepts.md#encryption-context) for the item\. If the DynamoDB encryption context doesn't include a value, such as the table name, that name\-value pair is omitted from the AWS KMS encryption context\.

1. The Direct KMS Provider derives a symmetric encryption key and a signing key from the data key\. By default, it uses [Secure Hash Algorithm \(SHA\) 256](https://en.wikipedia.org/wiki/SHA-2) and [RFC5869 HMAC\-based Key Derivation Function](https://tools.ietf.org/html/rfc5869) to derive a 256\-bit AES symmetric encryption key and a 256\-bit HMAC\-SHA\-256 signing key\. 

1. The Direct KMS Provider returns the output to the item encryptor\.

1. The item encryptor uses the encryption key to encrypt the specified attributes and the signing key to sign them, using the algorithms specified in the actual material description\. It removes the plaintext keys from memory as soon as possible\.

### Get decryption materials<a name="direct-kms-get-decryption-materials"></a>

This section describes in detail the inputs, outputs, and processing of the Direct KMS Provider when it receives a request for decryption materials from the [item encryptor](concepts.md#item-encryptor)\.

**Input ** \(from the application\)
+ The key ID of an AWS KMS key\. 

  The value of the key ID can be the key ID, key ARN, alias name or alias ARN of the AWS KMS key\. Any values that aren't included in the key ID, such as the Region, must be available in the [AWS named profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html)\. The key ARN provides all of the values that AWS KMS needs\.

**Input** \(from the item encryptor\)
+ A copy of the [DynamoDB encryption context](concepts.md#encryption-context) that contains the contents of the material description attribute\.

**Output** \(to the item encryptor\)
+ Encryption key \(plaintext\)
+ Signing key

**Processing**

1. The Direct KMS Provider gets the encrypted data key from the material description attribute in the encrypted item\. 

1. It asks AWS KMS to use the specified AWS KMS key to [decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) the encrypted data key\. The operation returns a plaintext key\.

   This request must use the same [AWS KMS encryption context](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context) that was used to generate and encrypt the data key\.
   + aws\-kms\-table – *table name*
   + *partition key name* – *partition key value* \(binary values are Base64\-encoded\)
   + \(Optional\) *sort key name* – *sort key value* \(binary values are Base64\-encoded\)
   + amzn\-ddb\-env\-alg – Encryption algorithm, by default AES/256
   + amzn\-ddb\-sig\-alg – Signing algorithm, by default HmacSHA256/256

1. The Direct KMS Provider uses [Secure Hash Algorithm \(SHA\) 256](https://en.wikipedia.org/wiki/SHA-2) and [RFC5869 HMAC\-based Key Derivation Function](https://tools.ietf.org/html/rfc5869) to derive a 256\-bit AES symmetric encryption key and a 256\-bit HMAC\-SHA\-256 signing key from the data key\. 

1. The Direct KMS Provider returns the output to the item encryptor\.

1. The item encryptor uses the signing key to verify the item\. If it succeeds, it uses the symmetric encryption key to decrypt the encrypted attribute values\. These operations use the encryption and signing algorithms specified in the actual material description\. The item encryptor removes the plaintext keys from memory as soon as possible\.