# Using the DynamoDB Encryption Client for Java<a name="java-using"></a>

This topic explains some of the features of the DynamoDB Encryption Client in Java that might not be found in other programming language implementations\. 

For details about programming with the DynamoDB Encryption Client, see the [Java examples](java-examples.md), the [examples](https://github.com/awslabs/aws-dynamodb-encryption-java/tree/master/examples) in the aws\-dynamodb\-encryption\-java repository on GitHub, and the [Javadoc](https://awslabs.github.io/aws-dynamodb-encryption-java/javadoc/) for the DynamoDB Encryption Client\.

**Topics**
+ [AttributeEncryptor](#attribute-encryptor)
+ [Attribute Actions in Java](#attribute-actions-java)

## AttributeEncryptor<a name="attribute-encryptor"></a>

The DynamoDB Encryption Client in Java has two [item encryptors](concepts.md#item-encryptor): the lower\-level [DynamoDBEncryptor](https://awslabs.github.io/aws-dynamodb-encryption-java/javadoc/com/amazonaws/services/dynamodbv2/datamodeling/encryption/DynamoDBEncryptor.html) and the [AttributeEncryptor](#attribute-encryptor)\. The `AttributeEncryptor` is a helper class that is designed to work with the [DynamoDBMapper](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.Methods.html) in the AWS SDK for Java\.

The `AttributeEncryptor` prevents you from encrypting the value of the primary key in your item\. It encrypts and signs all attributes except for the primary key attributes, which are signed but not encrypted\. If you explicitly tell the client to encrypt the primary key, it throws an exception\.

When you use the `AttributeEncryptor` with the `DynamoDBMapper`, it transparently encrypts and signs the item when you save it, and transparently verifies and decrypts the item when you load it\.

## Attribute Actions in Java<a name="attribute-actions-java"></a>

[Attribute actions](concepts.md#attribute-actions) determine which attribute values are encrypted and signed, which are only signed, and which are ignored\. The method you use to specify attribute actions depends on whether you use the `DynamoDBMapper` and `AttributeEncryptor`, or the lower\-level [DynamoDBEncryptor](https://awslabs.github.io/aws-dynamodb-encryption-java/javadoc/com/amazonaws/services/dynamodbv2/datamodeling/encryption/DynamoDBEncryptor.html)\.

### Attribute Actions for the DynamoDB Mapper<a name="attribute-action-java-mapper"></a>

When you use the `DynamoDBMapper` and `AttributeEncryptor` helper classes, you use annotations to specify the attribute actions\. The DynamoDB Encryption Client uses the [standard DynamoDB attribute annotations](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.Annotations.html) that define the attribute type to determine how to protect an attribute\. By default, all attributes are encrypted and signed except for primary keys, which are signed, but not encrypted\.

**Note**  
Do not encrypt the value of attributes with the [@DynamoDBVersionAttribute annotation](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.OptimisticLocking.html), although you can \(and should\) sign them\. Otherwise, conditions that use its value will have unintended effects\.

```
// Attributes are encrypted and signed
@DynamoDBAttribute(attributeName="Description")

// Partition keys are signed but not encrypted
@DynamoDBHashKey(attributeName="Title")

// Sort keys are signed but not encrypted
@DynamoDBRangeKey(attributeName="Author")
```

To specify exceptions, you use special encryption annotations that are defined in the DynamoDB Encryption Client for Java\. If you specify them at the class level, they become the default value for the class\.

```
// Sign only
@DoNotEncrypt

// Do nothing; not encrypted or signed
@DoNotTouch
```

For example, these annotations sign but do not encrypt the `PublicationYear` attribute, and do not encrypt or sign the `ISBN` attribute value\.

```
// Sign only (override the default)
@DoNotEncrypt
@DynamoDBAttribute(attributeName="PublicationYear")  

// Do nothing (override the default)
@DoNotTouch
@DynamoDBAttribute(attributeName="ISBN")
```

### Attribute Actions for the DynamoDBEncryptor<a name="attribute-action-default"></a>

To specify attribute actions when you use the [DynamoDBEncryptor](https://awslabs.github.io/aws-dynamodb-encryption-java/javadoc/com/amazonaws/services/dynamodbv2/datamodeling/encryption/DynamoDBEncryptor.html) directly, create a `HashMap` object in which the name\-value pairs represent attribute names and the specified actions\. 

The valid values are for the attribute actions are defined in the `EncryptionFlags` enumerated type\. You can use `ENCRYPT` and `SIGN` together, use `SIGN` alone, or omit both\. However, if you use `ENCRYPT` alone, the DynamoDB Encryption Client throws an error\. You cannot encrypt an attribute that you don't sign\.

```
ENCRYPT
SIGN
```

**Warning**  
Do not encrypt the primary key attributes\. They must remain in plaintext so DynamoDB can find the item without running a full table scan\.

If you specify a primary key in the encryption context and then specify `ENCRYPT` in the attribute action for either primary key attribute, the DynamoDB Encryption Client will throw an exception\.

For example, the following Java code creates an `actions` HashMap that encrypts and signs all attributes in the `record` item, except for the partition key and sort key attributes, which are signed but not encrypted, and the `test` attribute, which is not signed or encrypted\.

```
final EnumSet<EncryptionFlags> signOnly = EnumSet.of(EncryptionFlags.SIGN);
final EnumSet<EncryptionFlags> encryptAndSign = EnumSet.of(EncryptionFlags.ENCRYPT, EncryptionFlags.SIGN);
final Map<String, Set<EncryptionFlags>> actions = new HashMap<>();

for (final String attributeName : record.keySet()) {
  switch (attributeName) {
    case partitionKeyName: // no break; falls through to next case
    case sortKeyName:
      // Partition and sort keys must not be encrypted, but should be signed
      actions.put(attributeName, signOnly);
      break;
    case "test":
      // Don't encrypt or sign
      break;
    default:
      // Encrypt and sign everything else
      actions.put(attributeName, encryptAndSign);
      break;
  }
}
```

Then, when you call the [encryptRecord](https://awslabs.github.io/aws-dynamodb-encryption-java/javadoc/com/amazonaws/services/dynamodbv2/datamodeling/encryption/DynamoDBEncryptor.html#encryptRecord-java.util.Map-java.util.Map-com.amazonaws.services.dynamodbv2.datamodeling.encryption.EncryptionContext-) method of the `DynamoDBEncryptor`, specify the map as the value of the `attributeFlags` parameter\. For example, this call to `encryptRecord` uses the `actions` map\.

```
// Encrypt the plaintext record
final Map<String, AttributeValue> encrypted_record = encryptor.encryptRecord(record, actions, encryptionContext);
```