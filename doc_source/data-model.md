# Changing your data model<a name="data-model"></a>

Every time you encrypt or decrypt an item, you need to provide [attribute actions](concepts.md#attribute-actions) that tell the DynamoDB Encryption Client which attributes to encrypt and sign, which attributes to sign \(but not encrypt\), and which to ignore\. Attribute actions are not saved in the encrypted item and the DynamoDB Encryption Client does not update your attribute actions automatically\.

**Important**  
The DynamoDB Encryption Client does not support the encryption of existing, unencrypted DynamoDB table data\.

Whenever you change your data model, that is, when you add or remove attributes from your table items, you risk an error\. If the attribute actions that you specify do not account for all attributes in the item, the item might not be encrypted and signed the way that you intend\. More importantly, if the attribute actions that you provide when decrypting an item differ from the attribute actions that you provided when encrypting the item, the signature verification might fail\. 

For example, if the attribute actions used to encrypt the item tell it to sign the `test` attribute, the signature in the item will include the `test` attribute\. But if the attribute actions used to decrypt the item do not account for the `test` attribute, the verification will fail because the client will try to verify a signature that does not include the `test` attribute\. 

This is a particular problem when multiple applications read and write the same DynamoDB items because the DynamoDB Encryption Client must calculate the same signature for items in all applications\. It's also a problem for any distributed application because changes in attribute actions must propagate to all hosts\. Even if your DynamoDB tables are accessed by one host in one process, establishing a best practice process will help prevent errors if the project ever becomes more complex\.

To avoid signature validation errors that prevent you from reading your table items, use the following guidance\.
+ [Adding an attribute](#add-attribute) — If the new attribute changes your attribute actions, fully deploy the attribute action change before including the new attribute in an item\.
+ [Removing an attribute](#remove-attribute) — If you stop using an attribute in your items, do not change your attribute actions\. 
+ Changing the action — After you have used an attribute actions configuration to encrypt your table items, you cannot safely change the default action or the action for an existing attribute without re\-encrypting every item in your table\.

Signature validation errors can be extremely difficult to resolve, so the best approach is to prevent them\. 

**Topics**
+ [Adding an attribute](#add-attribute)
+ [Removing an attribute](#remove-attribute)

## Adding an attribute<a name="add-attribute"></a>

When you add a new attribute to table items, you might need to change your attribute actions\. To prevent signature validation errors, we recommend that you implement this change in a two\-stage process\. Verify that the first stage is complete before starting the second stage\.

1. Change the attribute actions in all applications that read or write to the table\. Deploy these changes and confirm that the update has been propagated to all destination hosts\. 

1. Write values to the new attribute in your table items\.

This two\-stage approach ensures that all applications and hosts have the same attribute actions, and will calculate the same signature, before any encounter the new attribute\. This is important even when the action for the attribute is *Do nothing* \(don't encrypt or sign\), because the default for some encryptors is to encrypt and sign\.

The following examples show the code for the first stage in this process\. They add a new item attribute, `link`, which stores a link to another table item\. Because this link must remain in plain text, the example assigns it the sign\-only action\. After fully deploying this change and then verifying that all applications and hosts have the new attribute actions, you can begin to use the `link` attribute in your table items\.

------
#### [ Java DynamoDB Mapper ]

When using the `DynamoDB Mapper` and `AttributeEncryptor`, by default, all attributes are encrypted and signed except for primary keys, which are signed but not encrypted\. To specify a sign\-only action, use the `@DoNotEncrypt` annotation\. 

This example uses the `@DoNotEncrypt` annotation for the new `link` attribute\.

```
@DynamoDBTable(tableName = "ExampleTable")
public static final class DataPoJo {
  private String partitionAttribute;
  private int sortAttribute;
  private String link;

  @DynamoDBHashKey(attributeName = "partition_attribute")
  public String getPartitionAttribute() {
    return partitionAttribute;
  }
    
  public void setPartitionAttribute(String partitionAttribute) {
    this.partitionAttribute = partitionAttribute;
  }

  @DynamoDBRangeKey(attributeName = "sort_attribute")
  public int getSortAttribute() {
    return sortAttribute;
  }

  public void setSortAttribute(int sortAttribute) {
    this.sortAttribute = sortAttribute;
  }

  @DynamoDBAttribute(attributeName = "link")
  @DoNotEncrypt
  public String getLink() {
    return link;
  }

  public void setLink(String link) {
    this.link = link;
  }

  @Override
  public String toString() {
    return "DataPoJo [partitionAttribute=" + partitionAttribute + ",
        sortAttribute=" + sortAttribute + ",
        link=" + link + "]";
  }
}
```

------
#### [ Java DynamoDB encryptor ]

 In the lower\-level DynamoDB encryptor, you must set actions for each attribute\. This example uses a switch statement where the default is `encryptAndSign` and exceptions are specified for the partition key, sort key, and the new `link` attribute\. In this example, if the link attribute code was not fully deployed before it was used, the link attribute would be encrypted and signed by some applications, but only signed by others\.

```
for (final String attributeName : record.keySet()) {
    switch (attributeName) {
        case partitionKeyName:
            // fall through to the next case
        case sortKeyName:
            // partition and sort keys must be signed, but not encrypted
            actions.put(attributeName, signOnly);
            break;
        case "link":
            // only signed
            actions.put(attributeName, signOnly);
            break;
        default:
            // Encrypt and sign all other attributes
            actions.put(attributeName, encryptAndSign);
            break;
    }
}
```

------
#### [ Python ]

In the DynamoDB Encryption Client for Python, you can specify a default action for all attributes and then specify exceptions\. 

If you use a Python [client helper class](python-using.md#python-helpers), you don't need to specify an attribute action for the primary key attributes\. The client helper classes prevent you from encrypting your primary key\. However, if you are not using a client helper class, you must set the SIGN\_ONLY action on your partition key and sort key\. If you accidentally encrypt your partition or sort key, you won't be able to recover your data without a full table scan\.

This example specifies an exception for the new `link` attribute, which gets the `SIGN_ONLY` action\.

```
actions = AttributeActions(
    default_action=CryptoAction.ENCRYPT_AND_SIGN,
    attribute_actions={
      'example': CryptoAction.DO_NOTHING,  
      'link': CryptoAction.SIGN_ONLY
    }
)
```

------

## Removing an attribute<a name="remove-attribute"></a>

If you no longer need an attribute in items that have been encrypted with the DynamoDB Encryption Client, you can stop using the attribute\. However, do not delete or change the action for that attribute\. If you do, and then encounter an item with that attribute, the signature calculated for the item will not match the original signature, and the signature validation will fail\.

Although you might be tempted to remove all traces of the attribute from your code, add a comment that the item is no longer used instead of deleting it\. Even if you do a full table scan to delete all instances of the attribute, an encrypted item with that attribute might be cached or in process somewhere in your configuration\.