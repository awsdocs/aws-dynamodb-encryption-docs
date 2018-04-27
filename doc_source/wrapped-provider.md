# Wrapped Materials Provider<a name="wrapped-provider"></a>

The Wrapped Materials Provider \(Wrapped CMP\) lets you use wrapping and signing keys from any source with the DynamoDB Encryption Client\. Unlike the Direct KMS Provider, the Wrapped CMP does not depend on any AWS service\. However, you must generate and manage your wrapping and signing keys outside of the client, including providing the correct keys to verify and decrypt the item\.

The Wrapped CMP generates a unique item encryption key for each item\. It wraps the item encryption key with the wrapping key that you provide and saves the wrapped item encryption key in the [Material Description attribute](concepts.md#material-description) of the item\. Because you supply the wrapping and signing keys, you can determine the extent to which the material is reused\.

The Wrapped CMP is a secure implementation and a good choice for applications that can manage cryptographic material\.

For example code, see \[BUGBUG\]\.

![\[The input, processing, and output of the Wrapped Materials Provider in the DynamoDB Encryption Client.\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/wrappedCMP.png)

The Wrapped CMP is one of several [cryptographic material provider](concepts.md#concept-material-provider) \(CMPs\) that the DynamoDB Encryption Client supports\. For information about the other CMPs, see [How to Choose a Cryptographic Materials Provider](crypto-materials-providers.md)\.

**Topics**
+ [Get Encryption Materials](#wrapped-cmp-get-encryption-materials)
+ [Get Decryption Materials](#wrapped-cmp-get-decryption-materials)

## Get Encryption Materials<a name="wrapped-cmp-get-encryption-materials"></a>

This section describes in detail the inputs outputs and processing of the Wrapped Materials Provider \(Wrapped CMP\) when it receives a request for encryption materials from the [item encryptor](http://junebl.aka.corp.amazon.com/workspaces/guides/src/AWSDynamoDBEncryptionDocs/build/server-root/dynamo-encryption-client/latest/devguide/concepts.html#item-encryptor)\. 

**Input** \(from application\)
+ Wrapping key\. An [Advanced Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) \(AES\) symmetric key, or an [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) public key\.
+ Unwrapping key: Optional and ignored\.
+ Signing key

**Output** \(to the item encryptor\):
+ Plaintext item encryption key
+ Signing key \(unchanged\)
+ [Descriptive material description](http://junebl.aka.corp.amazon.com/workspaces/guides/src/AWSDynamoDBEncryptionDocs/build/server-root/dynamo-encryption-client/latest/devguide/concepts.html#material-description): These values are saved in the Material Description attribute that the client adds to the item\. 
  + `amzn-ddb-env-key`: Base64\-encoded item encryption key encrypted by the AWS KMS customer master key
  + `amzn-ddb-env-alg`: Encryption algorithm that should be used with the item encryption key\. \(BUGBUG: Default?\)
  + `amzn-ddb-wrap-alg`: The wrapping algorithm that the Wrapped CMP used to wrap the item encryption key\. If the wrapping key is an AES key, the key is wrapped using unpadded `AES-Keywrap` as defined in [RFC 3394](https://tools.ietf.org/html/rfc3394.html)\. If the wrapping key is an RSA key, the key is wrapped using RSA OAEP with MGF1 padding\. 

**Processing**

When you encrypt an item, you pass in a wrapping key and a signing key\. An unwrapping key is optional and ignored\.

1. The Wrapped CMP generates a unique item encryption key for the table item\.

1. It uses the wrapping key that you specify to wrap the item encryption key\. Then, it removes it from memory as soon as possible\.

1. It returns the plaintext item encryption key, the signing key that you supplied, and a [descriptive material description](concepts.md#material-description) that includes the wrapped item encryption key, and the encryption and wrapping algorithms\.

1. The item encryptor uses the plaintext encryption key to encrypt the item\. It uses signing key that you supplied to sign the item\. Then, it removes the plaintext keys from memory as soon as possible\. It copies the fields in the descriptive material description, including the wrapped encryption key \(`amzn-ddb-env-key`\), to the [Material Description attribute](concepts.md#material-description) of the item\.

## Get Decryption Materials<a name="wrapped-cmp-get-decryption-materials"></a>

**Input** \(from application\)
+ Wrapping key\. Optional and ignored
+ Unwrapping key: The same [Advanced Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) \(AES\) symmetric key or [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) private key that corresponds to the RSA public key used to encrypt\.
+ Signing key

**Output** \(to the item encryptor\):
+ Plaintext item encryption key
+ Signing key \(unchanged\)

**Processing**

When you decrypt an item, you pass in an unwrapping key and a signing key\. An wrapping key is optional and ignored\.

1. The Wrapped CMP gets the wrapped item encryption key from the `Material Description` attribute of the item\.

1. It uses the unwrapping key and algorithm to unwrap the item encryption key\. 

1. It returns the plaintext item encryption key, the signing key, and encryption and signing algorithms to the item encryptor\.

1. The item encryptor uses the signing key to verify the item\. If it succeeds, it uses the item encryption key to decrypt the item\. Then it removes the plaintext keys from memory as soon as possible\.