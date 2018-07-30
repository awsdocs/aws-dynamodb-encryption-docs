# Example Code for the DynamoDB Encryption Client for Java<a name="java-examples"></a>

The following examples show you how to use the DynamoDB Encryption Client for Java to protect DynamoDB table items in your application\. You can find more examples \(and contribute your own\) in the [examples](https://github.com/awslabs/aws-dynamodb-encryption-python/tree/master/examples) directory of the [aws\-dynamodb\-encryption\-java](https://github.com/awslabs/aws-dynamodb-encryption-java/) repository on GitHub\.

**Topics**
+ [Using the DynamoDBEncryptor and Direct KMS Provider](#java-example-ddb-encryptor)

## Using the DynamoDBEncryptor and Direct KMS Provider<a name="java-example-ddb-encryptor"></a>

This example shows how to use the lower\-level [DynamoDBEncryptor](https://awslabs.github.io/aws-dynamodb-encryption-java/javadoc/com/amazonaws/services/dynamodbv2/datamodeling/encryption/DynamoDBEncryptor.html) with the [Direct KMS Provider](direct-kms-provider.md)\. The Direct KMS Provider generates and protects its cryptographic materials under an AWS Key Management Service \(AWS KMS\) [customer master key](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMK\) that you specify\.

You can use any compatible [cryptographic materials provider](concepts.md#concept-material-provider) \(CMP\) with the `DynamoDBEncryptor`, and you can use the Direct KMS Provider with the `DynamoDBMapper` and [AttributeEncryptor](java-using.md#attribute-encryptor)\.

**See the complete code sample**: [AwsKmsEncryptedItem\.java](https://github.com/awslabs/aws-dynamodb-encryption-java/blob/master/examples/com/amazonaws/examples/AwsKmsEncryptedItem.java)

Step 1: Create the Direct KMS Provider  
Create an instance of the AWS KMS client with the specified region\. Then, use the client instance to create an instance of the Direct KMS Provider with your preferred CMK\.   
This example uses the Amazon Resource Name \(ARN\) to identify the CMK, but you can use [any valid CMK identifier](http://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn)\.   

```
final String cmkArn = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
final String region = 'us-west-2'
      
final AWSKMS kms = AWSKMSClientBuilder.standard().withRegion(region).build();
final DirectKmsMaterialProvider cmp = new DirectKmsMaterialProvider(kms, cmkArn);
```

Step 2: Create an item  
This example defines a `record` HashMap that represents a sample table item\.  

```
final String partitionKeyName = "partition_attribute";
final String sortKeyName = "sort_attribute";

final Map<String, AttributeValue> record = new HashMap<>();
record.put(partitionKeyName, new AttributeValue().withS("value1"));
record.put(sortKeyName, new AttributeValue().withN("55"));
record.put("example", new AttributeValue().withS("data"));
record.put("numbers", new AttributeValue().withN("99"));
record.put("binary", new AttributeValue().withB(ByteBuffer.wrap(new byte[]{0x00, 0x01, 0x02})));
record.put("test", new AttributeValue().withS("test-value"));
```

Step 3: Create a DynamoDBEncryptor  
Create an instance of the `DynamoDBEncryptor` with the Direct KMS Provider\.  

```
final DynamoDBEncryptor encryptor = DynamoDBEncryptor.getInstance(cmp);
```

Step 4: Create a DynamoDB encryption context  
The [DynamoDB encryption context](concepts.md#encryption-context) contains information about the table structure and how it is encrypted and signed\. If you use the `DynamoDBMapper`, the `AttributeEncryptor` creates the encryption context for you\.  

```
final String tableName = "testTable";

final EncryptionContext encryptionContext = new EncryptionContext.Builder()
    .withTableName(tableName)
    .withHashKeyName(partitionKeyName)
    .withRangeKeyName(sortKeyName)
    .build();
```

Step 5: Create the attribute actions object  
[Attribute actions](concepts.md#attribute-actions) determine which attributes of the item are encrypted and signed, which are only signed, and which are not encrypted or signed\.  
In Java, to specify attribute actions, you create a HashMap of attribute name and `EncryptionFlags` value pairs\.   
For example, the following Java code creates an `actions` HashMap that encrypts and signs all attributes in the `record` item, except for the partition key and sort key attributes, which are signed, but not encrypted, and the `test` attribute, which is not signed or encrypted\.  

```
final EnumSet<EncryptionFlags> signOnly = EnumSet.of(EncryptionFlags.SIGN);
final EnumSet<EncryptionFlags> encryptAndSign = EnumSet.of(EncryptionFlags.ENCRYPT, EncryptionFlags.SIGN);
final Map<String, Set<EncryptionFlags>> actions = new HashMap<>();

for (final String attributeName : record.keySet()) {
  switch (attributeName) {
    case partitionKeyName: // fall through to the next case
    case sortKeyName:
      // Partition and sort keys must not be encrypted, but should be signed
      actions.put(attributeName, signOnly);
      break;
    case "test":
      // Neither encrypted nor signed
      break;
    default:
      // Encrypt and sign all other attributes
      actions.put(attributeName, encryptAndSign);
      break;
  }
}
```

Step 6: Encrypt and sign the item  
To encrypt and sign the table item, call the `encryptRecord` method on the instance of the `DynamoDBEncryptor`\. Specify the table item \(`record`\), the attribute actions \(`actions`\), and the encryption context \(`encryptionContext`\)\.  

```
final Map<String, AttributeValue> encrypted_record = encryptor.encryptRecord(record, actions, encryptionContext);
```

Step 7: Put the item in the DynamoDB table  
Finally, put the encrypted and signed item in the DynamoDB table\.  

```
final AmazonDynamoDB ddb = AmazonDynamoDBClientBuilder.defaultClient();
ddb.putItem(tableName, encrypted_record);
```