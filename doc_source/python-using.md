# Using the DynamoDB Encryption Client for Python<a name="python-using"></a>

This topic explains some of the features of the DynamoDB Encryption Client in Python that might not be found in other programming language implementations\. These features are designed to make it easier to use DynamoDB Encryption Client in the most secure way\. Unless you have an unusual use case, we recommend that you use them\.

For details about programming with the DynamoDB Encryption Client, see the [Python examples](python-examples.md) in this guide, the [examples](https://github.com/awslabs/aws-dynamodb-encryption-python/tree/master/examples) in the aws\-dynamodb\-encryption\-python repository on GitHub and the [Python documentation](http://aws-dynamodb-encryption-python.readthedocs.io/en/latest/) for the DynamoDB Encryption Client\.

**Topics**
+ [Client Helper Classes](#python-helpers)
+ [TableInfo Class](#table-info)
+ [Attribute actions in Python](#python-attribute-actions)

## Client Helper Classes<a name="python-helpers"></a>

The DynamoDB Encryption Client in Python includes several client helper classes that mirror the Boto3 classes for DynamoDB\. These helper classes are designed to make it easier to add encryption and signing to your existing DynamoDB application and avoid the most common problems\.
+ Prevent you from encrypting the primary key in your item, either by adding an override action for the primary key to the [AttributeActions](#python-attribute-actions) object, or by throwing an exception if your `AttributeActions` object explicitly tells the client to encrypt the primary key\. If the default action in your `AttributeActions` object is `DO_NOTHING`, the client helper classes use that action for the primary key\. Otherwise, they use `SIGN_ONLY`\.
+ Create a [TableInfo object](#python-helpers) and populate the [DynamoDB encryption context](concepts.md#encryption-context) based on a call to DynamoDB\. This helps to assure that your DynamoDB encryption context is accurate and the client can identify the primary key\.
+ Support `put_item` and `get_item` methods that transparently encrypt and sign your table items before saving them in your DynamoDB table\.

You can use the client helper classes instead of interacting directly with the lower\-level [item encryptor](concepts.md#item-encryptor)\. Use these classes unless you need to set advanced options in the item encryptor\.

The client helper classes include:
+ [EncryptedTable](https://github.com/awslabs/aws-dynamodb-encryption-python/blob/master/src/dynamodb_encryption_sdk/encrypted/table.py) for applications that use the [Table](http://boto3.readthedocs.io/en/latest/reference/services/dynamodb.html#table) resource in DynamoDB to process one table at a time\.
+ [EncryptedResource](https://github.com/awslabs/aws-dynamodb-encryption-python/blob/master/src/dynamodb_encryption_sdk/encrypted/resource.py) for applications that use the [Service Resource](http://boto3.readthedocs.io/en/latest/reference/services/dynamodb.html#service-resource) class in DynamoDB for batch processing\.
+ [EncryptedClient](https://github.com/awslabs/aws-dynamodb-encryption-python/blob/master/src/dynamodb_encryption_sdk/encrypted/client.py) for applications that use the [lower\-level client](http://boto3.readthedocs.io/en/latest/reference/services/dynamodb.html#client) in DynamoDB\.

To use the client helper classes, the caller must have permission to call the DynamoDB [DescribeTable](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_DescribeTable.html) operation on the target table\.

## TableInfo Class<a name="table-info"></a>

The [TableInfo](https://github.com/awslabs/aws-dynamodb-encryption-python/blob/master/src/dynamodb_encryption_sdk/structures.py) class is a helper class that represents a DynamoDB table, complete with fields for its primary key and secondary indexes\. It helps you to get accurate, real\-time information about the table\.

If you use a [client helper class](#python-helpers), it creates and uses a `TableInfo` object for you\. Otherwise, you can create one explicitly\. For an example, see [Use the Item Encryptor \(without client helper classes\)](python-examples.md#python-example-item-encryptor)\.

When you call the `refresh_indexed_attributes` method on a `TableInfo` object, it populates the property values of the object by calling the DynamoDB [DescribeTable](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_DescribeTable.html) operation\. Querying the table is much more reliable than hard\-coding index names\. The `TableInfo` class also includes an `encryption_context_values` property that includes the required values for the [DynamoDB encryption context](concepts.md#encryption-context)\. 

To use the `refresh_indexed_attributes` method, the caller must have permission to call the DynamoDB [DescribeTable](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_DescribeTable.html) operation on the target table\.

## Attribute actions in Python<a name="python-attribute-actions"></a>

[Attribute actions](concepts.md#attribute-actions) tell the item encryptor which actions to perform on each attribute of the item\. To specify attribute actions in Python, you create an `AttributeActions` object with a default action and any exceptions for particular attributes\. The valid values are defined in the `CryptoAction` enumerated type\.

```
DO_NOTHING = 0
SIGN_ONLY = 1
ENCRYPT_AND_SIGN = 2
```

For example, this `AttributeActions` object establishes `ENCRYPT_AND_SIGN` as the default for all attributes, and specifies exceptions for the `ISBN` and `PublicationYear` attributes\.

```
    actions = AttributeActions(
    default_action=CryptoAction.ENCRYPT_AND_SIGN,
    attribute_actions={
        'ISBN': CryptoAction.DO_NOTHING,
        'PublicationYear': CryptoAction.SIGN_ONLY
    }
)
```

If you use a [client helper class](#python-helpers), you do not need to specify an attribute action for the primary key\. The client helper classes prevent you from encrypting your primary key\.

If you do not use a client helper class and the default action is `ENCRYPT_AND_SIGN`, you must specify an action for the primary key\. The recommended action for primary keys is `SIGN_ONLY`\. To make this easy, use the `set_index_keys` method, which uses SIGN\_ONLY for primary keys, or DO\_NOTHING, when that is the default action\.

**Warning**  
If you encrypt your primary key, DynamoDB will not be able to find and return the item\. You will not be able to recover the item unless you perform a full scan of the table\. 

```
actions = AttributeActions(
        default_action=CryptoAction.ENCRYPT_AND_SIGN,
)
actions.set_index_keys(*table_info.protected_index_keys())
```