# Static Materials Provider<a name="static-provider"></a>

The Static Materials Provider \(Static CMP\) is a very simple CMP that is intended only for testing and proof\-of\-concept demonstrations\. It is not recommended for production environments\.

To use the Static CMP to encrypt a table item, you supply an [Advanced Encryption Standard](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) \(AES\) symmetric encryption key and a signing key or key pair\. You must supply the same keys to decrypt the encrypted item\. The Static CMP does not perform any cryptographic operations\. Instead, passes the encryption keys that you supply to the item encryptor unchanged\. The item encryptor encrypts the items directly under the encryption key\. Then, It uses the signing key directly to sign them\. 

Because the Static CMP does not generate any unique cryptographic material, all table items that you process are encrypted under the same encryption key and signed by the same signing key\. When you use the same key to encrypt the attributes values in numerous items or use the same key or key pair to sign all items, you risk exceeding the cryptographic limits of the keys\. 

![\[The input, processing, and output of the Static Materials Provider in the DynamoDB Encryption Client.\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/staticCMP.png)

**Note**  
The Asymmetric Static Provider in the Java library is a legacy provider that calls the [Wrapped CMP](wrapped-provider.md) with the encryption and signing key that you supply\. It uses static wrapping key and signing keys, but generates a unique encryption key for each item\. This provider is appropriate for use in production\. However, when possible, use the Wrapped CMP instead of this provider\.

The Static CMP is one of several [cryptographic material provider](concepts.md#concept-material-provider) \(CMPs\) that the DynamoDB Encryption Client supports\. For information about the other CMPs, see [How to Choose a Cryptographic Materials Provider](crypto-materials-providers.md)\.

**Topics**
+ [Get Encryption Materials](#static-cmp-get-encryption-materials)
+ [Get Decryption Materials\.](#static-cmp-get-decryption-materials)

## Get Encryption Materials<a name="static-cmp-get-encryption-materials"></a>

**Inputs** \(from the application\)
+ An encryption key\. This must be a symmetric key, such as an [Advanced Encryption Standard](https://tools.ietf.org/html/rfc3394.html) \(AES\) key\. 
+ A signing key\. This can be a symmetric key or an asymmetric key pair\. 

**Outputs** \(to the item encryptor\)
+ The encryption key passed as input\.
+ The signing key passed as input
+ Descriptive material description: None

## Get Decryption Materials\.<a name="static-cmp-get-decryption-materials"></a>

Although it includes separate methods for getting encryption materials and getting decryption materials, the behavior is the same\. 

**Inputs** \(from the application\)
+ An encryption key\. This must be a symmetric key, such as an [Advanced Encryption Standard](https://tools.ietf.org/html/rfc3394.html) \(AES\) key\. 
+ A signing key\. This can be a symmetric key or an asymmetric key pair\. 

**Outputs** \(to the item encryptor\)
+ The encryption key passed as input\.
+ The signing key passed as input