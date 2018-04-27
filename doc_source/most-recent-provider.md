# Most Recent Provider<a name="most-recent-provider"></a>

The *Most Recent Provider* is a variation of the [Wrapped Materials Provider](wrapped-provider.md) \(Wrapped CMP\) that uses a unique encryption key for each item, but can reuse the wrapping key that protects the encryption key, and the signing key that is used to sign the table item\. This [cryptographic materials provider](concepts.md#concept-material-provider) \(CMP\) is designed for applications that can benefit from caching and reusing wrapping and signing keys without violating their security requirements\.

The Most Recent Provider always generates a unique item encryption key for each table item\. Only the wrapping key and signing keys are reused\. And, you can add logic to your application to [rotate the wrapping and signing keys](#most-recent-provider-rotate) periodically\.

You can use encryption material from any source to protect the saved wrapping and signing keys, including keys that you supply or an [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) [customer master key](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys)\. And, because the Most Recent Provider caches wrapping and signing keys, it does not need to perform cryptographic operations or call a cryptographic service every time you encrypt or decrypt a table item\.

For examples, see \[BUGBUG\]\.

**Topics**
+ [About the Most Recent Provider](#about-mrp)
+ [About the MetaStore](#about-metastore)
+ [Get Encryption Material](#most-recent-provider-encrypt)
+ [Get Decryption Material](#most-recent-provider-decrypt)
+ [Rotating Cryptographic Material](#most-recent-provider-rotate)

## About the Most Recent Provider<a name="about-mrp"></a>

The Most Recent Provider gets a [Wrapped Materials Provider](wrapped-provider.md) \(Wrapped CMP\), and the wrapping, unwrapping, and signing keys that it requires, from a *MetaStore* that stores multiple encrypted versions of the provider's wrapping and signing keys\. Then it uses the Wrapped CMP to generate the cryptographic material it returns\. The Wrapped CMP generates a new item encryption key for every item\. Only the wrapping and signing keys are reused\.

The Most Recent Provider saves versions of the Wrapped CMP and corresponding plaintext wrapping and signing keys in a local Least Recently Used \(LRU\) cache in memory\. The cache enables the Most Recent Provider to get the Wrapped CMP and keys that it needs without calling the MetaStore every time\.

To request a Wrapped CMP from the MetaStore, the Most Recent Provider supplies its name and the version of the cryptographic materials it wants to use\. For encryption materials, the Most Recent Provider always requests the most recent version\. By default, that version number does not change, but you can force the Most Recent Provider to [rotate the wrapping and signing keys](#most-recent-provider-rotate) periodically\.

![\[A Most Recent Provider\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/most-recent-provider-1.png)

## About the MetaStore<a name="about-metastore"></a>

A *MetaStore* is a [provider store](concepts.md#provider-store) that creates and returns Wrapped CMPs\. It also returns reusable cryptographic materials for the Wrapped CMP, that is, the wrapping key, unwrapping key, and signing key that the Wrapped CMP requires\. 

The MetaStore generates the wrapping and signing keys, and then stores them \(in encrypted form\) in an internal DynamoDB table\. The materials in the table are protected by an internal DynamoDB Encryption Client, including an item encryptor and internal [cryptographic material provider](concepts.md#concept-material-provider) \(CMP\)\.

You can use any type of internal CMP in your MetaStore, including a compatible custom CMP or Wrapped CMP with cryptographic materials that you provide\. If the internal CMP in your MetaStore is a [Direct KMS Provider](wrapped-provider.md), your reusable wrapping and signing keys are protected under your [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) [customer master key](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys)\. A MetaStore can interact with multiple CMPs, but each Most Recent Provider interacts with only one instance of a MetaStore\.

![\[A Most Recent Provider\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/most-recent-provider-2.png)

## Get Encryption Material<a name="most-recent-provider-encrypt"></a>

**Inputs** \(from the application\): None

**Inputs** \(from the item encryptor\): [DynamoDB encryption context](concepts.md#encryption-context)

**Outputs** \(to the item encryptor\):

The outputs are the product of the [Wrapped CMP](wrapped-provider.md) in the Most Recent Provider\.
+ Plaintext item encryption key
+ Signing key
+ [Descriptive material description](http://junebl.aka.corp.amazon.com/workspaces/guides/src/AWSDynamoDBEncryptionDocs/build/server-root/dynamo-encryption-client/latest/devguide/concepts.html#material-description): These values are saved in the Material Description attribute that the client adds to the item\. 
  + `amzn-ddb-env-key`: Base64\-encoded item encryption key encrypted by the AWS KMS customer master key
  + `amzn-ddb-env-alg`: Encryption algorithm that should be used with the item encryption key\. \(BUGBUG: Default?\)
  + `amzn-ddb-wrap-alg`: The wrapping algorithm that the Wrapped CMP used to wrap the item encryption key\. If the wrapping key is an AES key, the key is wrapped using unpadded `AES-Keywrap` as defined in [RFC 3394](https://tools.ietf.org/html/rfc3394.html)\. If the wrapping key is an RSA key, the key is wrapped using RSA OAEP with MGF1 padding\. 
  + `amzn-ddb-meta-id`: The name of the Most Recent Provider and the version number of the encryption materials\. The format is 'name\#version', for example, 'mrp\-test\#5'\. The MetaStore adds this field to the descriptive material description\.

**Process**

The Most Recent Provider uses the following process to get the encryption materials that it returns to the item encryptor\. The process is illustrated in this diagram\.

![\[The input, processing, and output of the static provider in the DynamoDB Encryption Client.\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/most-recent-provider.png)

When you create a Most Recent Provider, you specify a MetaStore, a name for the Most Recent Provider, and a time\-to\-live value that determines how often the Most Recent Provider asks the MetaStore for the version number of its most recent version of cryptographic materials\. The MetaStore consists of a DynamoDB table, an item encryptor, and an internal CMP that you create and configure\.

**Note**  
When the time\-to\-live value expires, the Most Recent Provider asks the MetaStore for the maximum version number assigned to materials for that provider\. This action does not increment the version or cause the MetaStore to create new encryption materials\. For more information, see [Rotating Cryptographic Material](#most-recent-provider-rotate)

When the item encryptor asks the Most Recent Provider for encryption materials, the Most Recent Provider begins by searching its cache for the latest version of a [Wrapped CMP](wrapped-provider.md) and cryptographic materials\. 
+ If it finds the latest version of the Wrapped CMP and cryptographic material in its cache, the Most Recent Provider uses the Wrapped CMP to generate cryptographic material\. Then, it returns the cryptographic material to the item encryptor\. This operation does not require a call to the MetaStore or any other CMP\.

   
+ If the latest version of the Wrapped CMP and cryptographic material are not in its cache, the Most Recent Provider requests a Wrapped CMP and cryptographic materials from its MetaStore\. The request include the Most Recent Provider sends and the latest version number that it knows\.

   

  1. The MetaStore searches its internal DynamoDB table for cryptographic material, using the Most Recent Provider name as the partition key and the version number as the sort key\.
     + If the cryptographic materials are in the table, the MetaStore uses its internal item encryptor and CMP to decrypt them\. If the internal CMP is a [Direct KMS Provider](direct-kms-provider.md), this step includes a call to the [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\)\.

        
     + If the cryptographic materials are not in the table, the MetaStore generates new cryptographic material\. It uses its internal item encryptor and CMP to encrypt the cryptographic material, and then it stores the encrypted copy of the cryptographic material in the table\. The partition key is the name of the Most Recent Provider and the sort key is the requested version number\. If the internal CMP is a [Direct KMS Provider](direct-kms-provider.md), this step includes a call to the [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\)\.

  1. The MetaStore generates a Wrapped CMP that uses the cryptographic material\. It returns the Wrapped CMP with plaintext wrapping and signing keys to the Most Recent Provider\.

  1. The Most Recent Provider caches the Wrapped CMP and plaintext wrapping and signing keys in memory\. 

  1. The Most Recent Provider uses the Wrapped CMP to generate encryption materials\. Then, it returns the encryption materials to the item encryptor\.

## Get Decryption Material<a name="most-recent-provider-decrypt"></a>

**Inputs** \(from the item encryptor\): [DynamoDB encryption context](concepts.md#encryption-context)

**Outputs** \(to the item encryptor\):

The outputs are the product of the Wrapped CMP in the Most Recent Provider\.
+ Plaintext item encryption key
+ Signing key

**Process**

When the item encryptor asks the Most Recent Provider for decryption material, the Most Recent Provider begins by searching its cache for the version of a [Wrapped CMP](wrapped-provider.md) and cryptographic materials that were used to wrap the item encryption key and sign the item\.
+ If it finds the matching version of the Wrapped CMP and cryptographic material in its cache, the Most Recent Provider uses the Wrapped CMP to generate decryption material\. Then, it returns the decryption material to the item encryptor\. This operation does not require a call to the MetaStore or any other CMP\.
+ If the matching version of the Wrapped CMP and cryptographic material are not in its cache, the Most Recent Provider requests a Wrapped CMP and cryptographic materials from its MetaStore\. It sends its name and the matching version number in the request\.

  1. The MetaStore searches its internal DynamoDB table for cryptographic material, using the Most Recent Provider name as the partition key and the version number as the sort key\.
     + If the name and version number are not in the table, the MetaStore throws an exception\. If the MetaStore was used to generate the Wrapped CMP and cryptographic materials, they should be in an item its DynamoDB table, unless the item was intentionally deleted\.
     + If the name and version number are in the table:

       1. The MetaStore gets the encrypted cryptographic materials with the specified name and version number\.

       1. It uses its internal CMP to decrypt the encrypted cryptographic material\. If the internal CMP is a [Direct KMS Provider](direct-kms-provider.md), this step includes a call to the [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\)\.

       1. The MetaStore generates a Wrapped CMP that uses the cryptographic material\. It returns the Wrapped CMP with plaintext wrapping and signing keys to the Most Recent Provider\.

  1. The Most Recent Provider caches the Wrapped CMP and plaintext wrapping and signing keys in memory\. 

  1. The Most Recent Provider uses the Wrapped CMP to generate decryption materials\. Then, it returns the decryption materials to the item encryptor\.

## Rotating Cryptographic Material<a name="most-recent-provider-rotate"></a>

The Most Recent Provider returns a unique item encryption key for every item, but by default, it uses the same wrapping key and returns the same signing key for every item\. However, you can add logic that forces the Most Recent Provider to use a new wrapping key and return a new signing key\. This *key rotation* is important when you are encrypting extremely large numbers of table items\.

When you create a Most Recent Provider, you specify a time\-to\-live value that determines how often the Most Recent Provider asks its MetaStore for the maximum version number assigned to its cryptographic material\. The Most Recent Provider uses that maximum version number in its cache searches and MetaStore requests\. However, it never increments the version number\. And, because the Most Recent Provider never asks for materials with a higher version number, the MetaStore never generates new cryptographic material for the Most Recent Provider\.

```
# MostRecentProvider
if (time > ttl) { 
    max = get-maximum-version(provider-name)
}

# MetaStore
return 0

# Most Recent Provider
get-or-create(provider-name, 0)
```

However, your application can call the MetaStore's Create New Provider operation directly with the name of the Most Recent Provider\. As a result, the MetaStore generates new cryptographic material for the Most Recent Provider and saves an encrypted copy it in its internal table with a higher version number\. \(It will also return a wrapped CMP with cryptographic material; you can delete it\.\) The next time the Most Recent Provider asks the MetaStore for its maximum version number, it gets the new higher version number, and uses it in subsequent requests\.

```
# Application
create-new-provider(provider-name)

# MetaStore creates/returns new Wrapped CMP + materials
version += 1

# MostRecentProvider
if (time > ttl) { 
    max = get-maximum-version(provider-name)
}

# MetaStore
return 1

# Most Recent Provider
get-or-create(provider-name, 1)
```

You can schedule your Create New Provider calls based on time, the number of items or attributes processed, or any other metric that makes sense for your application\.