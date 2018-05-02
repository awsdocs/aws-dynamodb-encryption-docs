# Client\-Side or Server\-Side Encryption?<a name="client-server-side"></a>

The DynamoDB Encryption Client supports *client\-side encryption*, where you encrypt your table data before you send it to DynamoDB\. However, DynamoDB supports an *encryption at rest* feature that transparently encrypts your table when it is persisted to disk and decrypts it when you access the table\. 

The tools that you choose depend on the sensitivity of your data and the security requirements of your application\. 

You can also use the DynamoDB Encryption Client and encryption at rest together\. When you send encrypted and signed items to DynamoDB, DynamoDB doesn't recognize the items as being protected\. It just detects typical table items with binary attribute values\. Those table items can be part of an encrypted or unencrypted table\.

**Encryption at Rest**

DynamoDB offers [encryption at rest](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EncryptionAtRest.html), a *server\-side encryption* option in which DynamoDB transparently encrypts your tables for you when the table is persisted to disk, and decrypts them when you access the table data\. 
+ **It's easy to use\.** Just select the encryption option when you create a table\. \(You cannot change the option on an existing table\.\) DynamoDB transparently encrypts and decrypts the table for you\.
+ **DynamoDB creates and manages the cryptographic keys\. **The unique key for each table is protected by an [AWS Key Management Service](http://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) customer master key that never leaves AWS KMS unencrypted\.
+ **All table data is encrypted on disk\. **When an encrypted table is saved to disk, DynamoDB encrypts all table data, including the primary key and local and global secondary indexes\. \(If a table partition is split, the sort key is stored in plaintext in the table metadata\.\)
+ **Your items are decrypted when you access them\. **When you access the table, DynamoDB decrypts the part of the table that includes your target item, and returns the plaintext item to you\.

 

**DynamoDB Encryption Client**

Client\-side encryption is designed to protect your data at its source\. Your plaintext data is never exposed to any third party, including AWS\. However, you need to add the encryption features to your DynamoDB applications\. 
+ **Your data is protected in transit and at rest\.** It is never exposed to any third party, including AWS\.
+ **You choose how your cryptographic keys are generated and protected\.** You can create and manage them yourself, or use a cryptographic service such as AWS Key Management Service or [AWS CloudHSM](http://docs.aws.amazon.com/cloudhsm/latest/userguide/) to generate and protect your keys\.
+ **You determine how your data is protected by [selecting a cryptographic materials provider](crypto-materials-providers.md) \(CMP\), or writing one of your own\. **The CMP determines the encryption strategy used, including when unique keys are generated, and the encryption and signing algorithms that are used\.
+ **The DynamoDB Encryption Client doesn't encrypt the entire table\.** It encrypts the values of [specified attributes](concepts.md#attribute-actions) in a table item, but not the attribute names\. By default, it doesn't encrypt the names or values of the primary key \(partition key and sort key\)\. It also signs the item so you can detect unauthorized changes to the item as whole\.

 

**AWS Encryption SDK**

If you are encrypting data that you store DynamoDB, we recommend the DynamoDB Encryption Client\. 

The [AWS Encryption SDK](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/) is a client\-side encryption library that helps you to encrypt and decrypt generic data\. Although it can protect any type of data, it isn't designed to work with structured data, like database records\. Unlike the DynamoDB Encryption Client, the AWS Encryption SDK cannot provide item\-level integrity checking and it has no logic to recognize attributes or prevent encryption of primary keys\.

If you use the AWS Encryption SDK to encrypt any element of your table, remember that it isn't compatible with the DynamoDB Encryption Client\. You cannot encrypt with one library and decrypt with the other\.