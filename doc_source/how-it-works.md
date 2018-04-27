# How the DynamoDB Encryption Client Works<a name="how-it-works"></a>

The DynamoDB Encryption Client is designed especially to protect the data that you use in DynamoDB\. Most components are represented by abstract classes so you can create your own custom implementations, but the libraries include secure implementations that you can extend or use unchanged\.

**Encrypting and Signing Table Items**

The item encryptor in the DynamoDB Encryption Client and cryptographic materials manager that you choose work together to generate encryption materials, and then use them to encrypt and sign your table items\. The following graphic shows a high\-level view of this process\.

![\[Encrypting and Signing Items in the DynamoDB Encryption Client\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/arch-encrypt.png)

At the core of the client is the [item encryptor](concepts.md#item-encryptor), which encrypts and signs the item\. You can interact with the item encryptor directly or use helper classes that implement secure default behavior\. 

The item encryptor gets the encryption materials, including encryption and signing keys, from a [cryptographic material provider](concepts.md#concept-material-provider) \(CMP\)\. The CMP also determines the encryption strategy, including the encryption algorithms, how encryption keys are generate and managed, and whether cryptographic materials generated for each item or reused\. You can write your own CMP or use one of the CMPs available for your programming language\.

While assembling the requested keys, the CMP saves important data, including the type of keys returns and the encryption and signing algorithms to be used, in a [descriptive material description](concepts.md#material-description) that it returns to the item encryptor with the cryptographic material\.

When you configure your client instance, you provide the following inputs for the item encryptor\. The helper classes generate some of the inputs for you\.
+ An instance of the CMP that you want to use; configured with your preferences\.
+ An [attribute actions](concepts.md#attribute-actions) object that specifies which attribute values to encrypt and which attributes to sign\. 
+ A [DynamoDB encryption context](concepts.md#encryption-context) that describes the table and the item, including the names of its primary key attributes\. \(This encryption context is not related to the encryption context in AWS KMS or the AWS Encryption SDK, despite the similar name\.\)

Then, when you ask the client to encrypt and sign a plaintext item, the item encryptor gets the encryption materials from the instance of the CMP\. Guided by the instructions in the attribute actions and DynamoDB encryption context, it encrypts and signs the item, and adds attributes that store the descriptive material description and the item signature\. The result is an encrypted and signed item\.

If you call the helper class operation that adds the item to a DynamoDB table, the item is transparently encrypted and signed, as shown, then put in the DynamoDB table that you specify\. 

**Verifying and Decrypting Table Items**

The item encryptor and cryptographic materials manager also work together to collect and use the keys to verify and decrypt your item, as shown in the following diagram\.

![\[Verifying and Decrypting Items in the DynamoDB Encryption Client\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/arch-decrypt.png)

When you configure a client instance to verify and decrypt a table item, you provide the following inputs for the item encryptor\. The helper classes generate some of the inputs for you\.
+ The same CMP that you used to encrypt and sign the item \(or an instance with the same configuration\)\. The encrypted item does not have any value that identifies the CMP that was used to generate its cryptographic materials\.
+ An [attribute actions](concepts.md#attribute-actions) object that specifies which attribute values were encrypted and which attributes were signed\. 
+ A [DynamoDB encryption context](concepts.md#encryption-context) that describes the table and the item, including the names of its primary key attributes\. \(This encryption context is not related to the encryption context in AWS KMS or the AWS Encryption SDK, despite the shared name\.\)

Then, when you ask the client to verify and decrypt an encrypted item, the item encryptor gets the decryption materials from the [cryptographic material provider](concepts.md#concept-material-provider) \(CMP\)\. The CMP uses the descriptive material description in the item attribute to assemble the correct material\. 

Guided by the instructions in the attribute actions and DynamoDB encryption context, the item encryptor tries to verify the item\. If verification fails, the item encryptor throws an exception and the call to decrypt the item fails\. If the verification succeeds, the item encryptor decrypts the encrypted attribute values and returns the plaintext item\.

If you call the helper class operation that gets an encrypted item from a DynamoDB table, the item is transparently verified and decrypted, as shown, then returned to the caller\. 