# Direct KMS Materials Provider<a name="direct-kms-provider"></a>

The Direct KMS Provider \(Direct KMS\) protects your table items under an [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) that never leaves AWS KMS unencrypted\. This cryptographic materials provider returns a unique encryption key and signing key for every table item\. To do so, it calls AWS KMS every time you encrypt or decrypt an item\.

For example code, see \[BUGBUG\]\.

![\[The input, processing, and output of the Direct KMS Provider in the DynamoDB Encryption Client.\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/directKMS.png)
+ To generate encryption materials, the Direct KMS Provider asks AWS KMS to [generate a unique data key](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) using a [customer master key](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMK\) that you specify\. It derives encryption and signing keys for the item from the plaintext copy of the [data key](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys), and then returns the encryption and signing keys, along with the encrypted data key, which is stored in the [Material Description attribute](concepts.md#material-description) of the item\. 

  The item encryptor uses the encryption and signing keys and removes them from memory as soon as possible\. Only the encrypted data key from which they were derived is saved with the encrypted and signed item\.
+ To generate decryption materials, the Direct KMS Provider asks AWS KMS to use the CMK that you specify to decrypt the encrypted data key\. Then, it derives and returns keys to verify the signature and decrypt the encrypted attribute values\.

  The item encryptor uses verifies the item and, if it succeeds, decrypts the encrypted values\.

The Direct KMS Provider calls AWS KMS every time it generates encryption or decryption materials\. If you are processing DynamoDB items at a high frequency and large scale, you might exceed the AWS KMS [requests\-per\-second limit](https://alpha-docs-aws.amazon.com/kms/latest/developerguide/limits.html#requests-per-second), causing processing delays\.

To use the Direct KMS Provider, the caller must have [an AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) and at least one AWS KMS customer master key\. The caller must have permission to call the [GenerateDataKey](http://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) and [Decrypt](http://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operations on the customer master key\. The Direct KMS provider in the libraries does not let you specify a new CMK for each operation, but the classes include extension points that let you add this feature\.

The Wrapped CMP is one of several [cryptographic material provider](concepts.md#concept-material-provider) \(CMPs\) that the DynamoDB Encryption Client supports\. For information about the other CMPs, see [How to Choose a Cryptographic Materials Provider](crypto-materials-providers.md)\.

**Topics**
+ [Get Encryption Materials](#direct-kms-get-encryption-materials)
+ [Get Decryption Materials](#direct-kms-get-decryption-materials)

## Get Encryption Materials<a name="direct-kms-get-encryption-materials"></a>

This section describes in detail the inputs outputs and processing of the Direct KMS Provider when it receives a request for encryption materials from the [item encryptor](concepts.md#item-encryptor)\.

**Input ** \(from application\)
+ The key ID of an AWS KMS customer master key\. 

  The value of the key ID can be the ID or Amazon Resource Name \(ARN\) of the customer master key, or an alias or alias ARN, provided that any values that are omitted, such as the region, are available in the [AWS named profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html)\. The customer master key ARN provides all of the values that AWS KMS needs\.

**Output** \(to the item encryptor\)
+ Encryption key \(plaintext\)
+ Signing key
+ In [descriptive material description](concepts.md#material-description): These values are saved in the Material Description attribute that the client adds to the item\.
  + amzn\-ddb\-env\-key: Base64\-encoded data key encrypted by the AWS KMS customer master key
  + amzn\-ddb\-env\-alg: Encryption algorithm, by default [AES/256](https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines/archived-crypto-projects/aes-development)
  + amzn\-ddb\-sig\-alg: Signing algorithm, by default, [HmacSHA256/256](https://en.wikipedia.org/wiki/HMAC)

**Processing**

1. The Direct KMS provider sends AWS KMS a request to use the specified customer master key to [generate a unique data key](https://alpha-docs-aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) for the item\. The operation returns a plaintext key and a copy that is encrypted under the CMK\. This is known as the *initial key material*\.

   The request includes the following [AWS KMS encryption context](https://alpha-docs-aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context)\. These non\-secret values are cryptographically bound to the encrypted object, so the same encryption context is required on decrypt\. You can use these values to identify the call to AWS KMS in [AWS CloudTrail logs](https://alpha-docs-aws.amazon.com/kms/latest/developerguide/monitoring-overview.html)\.
   + aws\-kms\-table : *table name*
   + *partition key name* : *partition key value* \(binary values are Base64\-encoded\)
   + \(Optional\) *sort key name* : *sort key value* \(binary values are Base64\-encoded\)
   + amzn\-ddb\-env\-alg : \(Encryption algorithm, by default AES/256\)
   + amzn\-ddb\-sig\-alg : \(Signing algorithm, by default HmacSHA256/256\)

1. The Direct KMS Provider uses [Secure Hash Algorithm \(SHA\) 256](https://en.wikipedia.org/wiki/SHA-2) and [RFC5869 HMAC\-based Key Derivation Function](https://tools.ietf.org/html/rfc5869) to derive a 256\-bit AES symmetric encryption key and a 256\-bit HMAC signing key from the data key\. 

1. The Direct KMS Provider returns the output to the item encryptor\.

1. The item encryptor uses the encryption key to encrypt the specified attributes and the signing key to sign them, using the algorithms specified in the descriptive material description\. It removes the plaintext keys from memory as soon as possible\.

## Get Decryption Materials<a name="direct-kms-get-decryption-materials"></a>

This section describes in detail the inputs outputs and processing of the Direct KMS Provider when it receives a request for decryption materials from the [item encryptor](concepts.md#item-encryptor)\.

**Input ** \(from application\)
+ The key ID of an AWS KMS customer master key\. 

  The value of the key ID can be the ID or Amazon Resource Name \(ARN\) of the customer master key, or an alias or alias ARN, provided that any values that are omitted, such as the region, are available in the [AWS named profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html)\. The customer master key ARN provides all of the values that AWS KMS needs\.

**Output** \(to the item encryptor\)
+ Encryption key \(plaintext\)
+ Signing key

**Processing**

1. The Direct KMS provider gets the encrypted data key from the Material Description attribute in the encrypted item\. 

1. It asks AWS KMS to use the specified customer master key to [decrypt](https://alpha-docs-aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) the encrypted data key\. The operation returns a plaintext key\.

   This request must use the same [AWS KMS encryption context](https://alpha-docs-aws.amazon.com/kms/latest/developerguide/concepts.html#encrypt_context) that was used to generate and encrypt the data key\.
   + aws\-kms\-table : *table name*
   + *partition key name* : *partition key value* \(binary values are Base64\-encoded\)
   + \(Optional\) *sort key name* : *sort key value* \(binary values are Base64\-encoded\)
   + amzn\-ddb\-env\-alg : \(Encryption algorithm, by default AES/256\)
   + amzn\-ddb\-sig\-alg : \(Signing algorithm, by default HmacSHA256/256\)

1. The Direct KMS Provider uses [Secure Hash Algorithm \(SHA\) 256](https://en.wikipedia.org/wiki/SHA-2) and [RFC5869 HMAC\-based Key Derivation Function](https://tools.ietf.org/html/rfc5869) to derive a 256\-bit AES symmetric encryption key and a 256\-bit HMAC signing key from the data key\. 

1. The Direct KMS Provider returns the output to the item encryptor\.

1. The item encryptor uses the signing key to verify the item\. If it succeeds, it uses the symmetric encryption key to decrypt the encrypted attribute values\. These operations use the encryption and signing algorithms specified in the descriptive material description\. The item encryptor removes the plaintext keys from memory as soon as possible\.