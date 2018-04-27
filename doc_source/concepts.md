# Amazon DynamoDB Encryption Client Concepts<a name="concepts"></a>

This topic explains the concepts and terminology used in the Amazon DynamoDB Encryption Client\. To learn how the components of the DynamoDB Encryption Client interact, see [How the DynamoDB Encryption Client Works](how-it-works.md)\.

**Topics**
+ [Cryptographic Materials Provider \(CMP\)](#concept-material-provider)
+ [Item Encryptor](#item-encryptor)
+ [Attribute actions](#attribute-actions)
+ [Delegated Keys](#delegated-keys)
+ [Material Description](#material-description)
+ [DynamoDB Encryption Context](#encryption-context)
+ [Provider store](#provider-store)

## Cryptographic Materials Provider \(CMP\)<a name="concept-material-provider"></a>

One of the first decisions that you will make when implementing the DynamoDB Encryption Client is to [select a cryptographic materials provider](crypto-materials-providers.md) \(also known as an *encryption materials provider*\. Your choice will determine much of the rest of the implementation\. 

A *cryptographic materials provider* \(CMP\) collects, assembles, and returns the cryptographic material that the [item encryptor](#item-encryptor) uses to encrypt and sign the your table items\. The CMP determines the encryption algorithm that are used and how encryption and signing keys are generate and protected\.

The CMPs interact with the [item encryptor](#item-encryptor)\. The item encryptor requests encryption or decryption materials from the CMP and the CMP returns the to the item encryptor\. Then, the item encryptor uses the cryptographic material to encrypt and sign, or verify and decrypt, the item\. For details about this interaction, see [How the DynamoDB Encryption Client Works](how-it-works.md)\.

You specify the CMP when you configure an encrypted table\. You can create a compatible custom CMP, or use one of the many CMPs in the library\. Most CMPs are available in multiple programming languages\. Although most applications uses the same CMP for all items in a table, you can choose a different CMP for each item\. 

## Item Encryptor<a name="item-encryptor"></a>

The *item encryptor* is a lower\-level component that performs cryptographic operations for the DynamoDB Encryption Client\. It requests cryptographic materials from a [cryptographic material provider](#concept-material-provider) \(CMP\), then uses the material that the CMP returns to encrypt and sign, or verify and decrypt, your table item\.

To perform cryptographic operations on raw keys, the item encryptor performs the operations directly\. On [delegated keys](#delegated-keys), it calls the methods that the delegated keys support\.

You can interact with the item encryptor directly or use the helper classes that your library provides\. For example, the DynamoDB Encryption Client in Java includes an `AttributeEncryptor` helper class that you can use instead of interacting directly with `DynamoDBEncryptor` item encryptor\. The Python library includes `EncryptedTable`, `EncryptedClient`, and `EncryptedResource` helper classes that interact with the item encryptor for you\.

## Attribute actions<a name="attribute-actions"></a>

*Attribute actions* tell the item encryptor which actions to perform on each attribute of the item\. 

In general, use `ENCRYPT_AND_SIGN` for all attributes that can store sensitive data\. For primary key attributes \(partition key and sort key\), use SIGN\_ONLY\. Some programming languages include helper classes that prevent you from encrypting your primary key\.

**Warning**  
If you encrypt your primary key, DynamoDB cannot find and retrieve the item\. To find the item, you need to do a full table scan\.

The attribute action values can be one of the following:
+ **Encrypt and sign**\. Encrypt the attribute value\. Include the attribute \(name and value\) in the item signature\.
+ **Sign only**: Include the attribute in the item signature\.
+ **Do Nothing**: Do not encrypt or sign the attribute\.

**Note**  
The [`MaterialDescription` attribute](#material-description) and signature attribute are neither signed nor encrypted\. You do not need to specify attribute actions for these attribute\.

The technique that you use to specify the attribute actions can be different for each programming language\. It might also be specific to helper classes that you use\.

For details, see the documentation for your programming language\.
+ [Python](python-using.md#python-attribute-actions)
+ [Java](java-using.md#attribute-actions-java)

## Delegated Keys<a name="delegated-keys"></a>

When you supply a cryptographic key to the DynamoDB Encryption Client, you can provide a raw key \(a Base64\-encoded byte array\), a *secret key* object \(Java\), or a *delegated key*, which is an abstract object that performs cryptographic operations\. Each delegated key usually represents one actual key type, such as an AES key or an RSA public key, and contains key material for that type of key\.

The DynamoDB Encryption Client includes a `JceNameLocalDelegatedKey` delegated key that supports the algorithms specified in the [Java Cryptography Architecture Standard Algorithm Names](https://docs.oracle.com/javase/7/docs/technotes/guides/security/StandardNames.html) document\.

To use a delegated key, the [item encryptor](#item-encryptor) or [cryptographic materials provider](#concept-material-provider) \(CMP\) calls the operations on the delegated key\. For example, when a CMP uses a delegated wrapping key, it calls the `wrap` operation of the delegated key, and passes in the item encryption key, which might also be a delegated key\. The `wrap` operation wraps the key material in the delegated item encryption key\.

Each delegated key supports some or all of the following operations\.
+ **algorithm**: Returns a value that identifies the algorithm\(s\) that the delegated supports\.
+ **encrypt**: Encrypts data
+ **decrypt**: Decrypts ciphertext
+ **wrap**: Encrypts an item encryption key
+ **unwrap**: Decrypts a wrapped item encryption key
+ **sign**: Calculates a signature over data
+ **verify**: Verifies a signature against data

## Material Description<a name="material-description"></a>

The *material description* for an encrypted table item consists of information about how the table item is encrypted and signed, such as encryption algorithms\. The [cryptographic materials provider](#concept-material-provider) \(CMP\) records the material description as it assembles the cryptographic materials for encryption and signing\. Later, when it needs to assemble cryptographic materials to verify and decrypt the item, it uses the material description as its guide\. 

In the DynamoDB Encryption Client, the material description refers to three related elements:

**Requested material description**  
Some [cryptographic materials providers](#concept-material-provider) \(CMPs\) let you specify advanced options, such as an encryption algorithm\. To indicate your choices, you add name\-value pairs to the material description property of the [DynamoDB encryption context](#encryption-context) in your request to encrypt a table item\. This element is known as the *requested material description*\. The valid values in the requested material description are defined by the CMP that you choose\.   
Because the material description can override best\-practice default values, we recommend that you omit the requested material description unless you have a compelling reason\.

**Descriptive material description**  
The material description that the [cryptographic materials provider](#concept-material-provider) \(CMP\) returns is known as the *descriptive material description*\. It describes the actual values that were used to assemble the cryptographic materials\. It usually consists of the requested material description with additions and changes\. The CMP stores the descriptive material description in the [DynamoDB encryption context](#encryption-context) that it returns to the item encryptor\. 

**Material Description attribute**  
The *descriptive material description* is stored in a `Material Description` attribute in the encrypted item\. The CMP uses the values in the attribute to generate the cryptographic material required to verify and decrypt the item\.

## DynamoDB Encryption Context<a name="encryption-context"></a>

The *DynamoDB encryption context* supplies information about the table and item to the [cryptographic materials provider](#concept-material-provider) \(CMP\)\. In advanced implementations, the DynamoDB encryption context can include a [requested material description](#material-description)\.

**Note**  
The *DynamoDB encryption context* in the DynamoDB Encryption Client is not related to the *encryption context* in AWS Key Management Service \(AWS KMS\) and the AWS Encryption SDK\. 

If you interact with the [item encryptor](#item-encryptor) directly, you need to provide a DynamoDB encryption context when you configure your CMP\. Most helper classes create the DynamoDB encryption context for you\.

The DynamoDB encryption context might vary with the CMP\. However, it typically consists of the following fields\.
+ Required values: 
  + Table name
  + Partition key name
  + Sort key name
+ Optional: 
  + Attribute name\-value pairs
  + [requested material description](#material-description)

## Provider store<a name="provider-store"></a>

A *provider store* is a component that returns [cryptographic materials providers](#concept-material-provider) \(CMPs\)\. The provider store can create the CMPs or get them from another source, such as another provider store\.

The DynamoDB Encryption Client includes a *MetaStore*, which is a provider store that creates Wrapped CMPs with reused wrapping and signing keys\. The [Most Recent Provider](most-recent-provider.md) gets its Wrapped CMPs from the MetaStore, but you can use the MetaStore to supply Wrapped CMPs to any component\.

See Also:
+ Provider store interface \([Java](https://github.com/awslabs/aws-dynamodb-encryption-java/blob/master/src/main/java/com/amazonaws/services/dynamodbv2/datamodeling/encryption/providers/store/ProviderStore.java), Python\)
+ MetaStore class \([Java](https://github.com/awslabs/aws-dynamodb-encryption-java/blob/master/src/main/java/com/amazonaws/services/dynamodbv2/datamodeling/encryption/providers/store/MetaStore.java), Python\)