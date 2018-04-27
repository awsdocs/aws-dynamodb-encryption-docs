# DynamoDB Encryption Client for Python Example Code<a name="python-examples"></a>

The following examples show you how to use the DynamoDB Encryption Client for Python to protect DynamoDB data in your application\. You can find more examples \(and contribute your own\) in the [examples](https://github.com/awslabs/aws-dynamodb-encryption-python/tree/master/examples) directory of the `aws-dynamodb-encryption-python` repository in GitHub\.

**Topics**
+ [Use the Item Encryptor \(without client helper classes\)](#python-example-item-encryptor)
+ [Use the Direct KMS Provider](#python-example-streams)
+ [Use the Wrapped Materials Provider](#python-example-multiple-providers)
+ [Use the Most Recent Provider](#python-example-mrp)

## Use the Item Encryptor \(without client helper classes\)<a name="python-example-item-encryptor"></a>

This example shows you how to interact directly with the [item encryptor](concepts.md#item-encryptor) in the DynamoDB Encryption Client when encrypting table items, instead of using the [client helper classes](python-using.md#python-helpers) that interact with the item encryptor for you\. 

When you use this technique, you create the DynamoDB encryption context and configuration object \(CryptoConfig\) manually\. Also, you encrypt the items in one call and put them in your DynamoDB table in a separate call\. This allows you to customize your `put_item` calls and use the DynamoDB Encryption Client to encrypt and sign structured data that is never sent to DynamoDB\.

This example uses the [Direct KMS Provider](direct-kms-provider.md), but you can use any compatible provider\.

**See the complete code sample**: [aws\_kms\_encrypted\_item\.py](https://github.com/awslabs/aws-dynamodb-encryption-python/blob/master/examples/src/aws_kms_encrypted_item.py)

Step 1: Create the table  
Start by creating an instance of a standard DynamoDB table with the table name\.  

```
table_name='test-table'
table = boto3.resource('dynamodb').Table(table_name)
```

Step 2: Create a cryptographic materials provider  
Create an instance of the [cryptographic materials provider](crypto-materials-providers.md) \(CMP\) that you selected\.  
This example uses the [Direct KMS Provider](direct-kms-provider.md), but you can use any compatible cryptographic materials provider\. To create a Direct KMS Provider, specify a [customer master key](http://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMK\)\. This example uses the Amazon Resource Name \(ARN\) of the CMK, but you can use any valid CMK identifier\.  

```
aws_cmk_id=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
aws_kms_cmp = AwsKmsCryptographicMaterialsProvider(key_id=aws_cmk_id)
```

Step 3: Use the TableInfo helper class  
To get information about the table from DynamoDB, create an instance of the [TableInfo](python-using.md#python-helpers) helper class\. When you work directly with the item encryptor, you need to create a `TableInfo` instance and call its methods\. The [client helper classes](python-using.md#python-helpers) do this for you\.  
The `refresh_indexed_attributes` method of `TableInfo` uses the [DescribeTable](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_DescribeTable.html) DynamoDB operation to get real\-time, accurate information about the table, including its primary key and its local and global secondary indexes\. The caller needs to have permission to call `DescribeTable`\.  

```
table_info = TableInfo(name=table_name)
table_info.refresh_indexed_attributes(table.meta.client)
```

Step 4: Create the DynamoDB encryption context  
The [DynamoDB encryption context](concepts.md#encryption-context) contains information about the table structure and how it is encrypted and signed\. This example creates a DynamoDB encryption context explicitly, because it interacts with the item encryptor\. The [client helper classes](python-using.md#python-helpers) create the DynamoDB encryption context for you\.   
To get the partition key and sort key, you can use the properties of the [TableInfo](python-using.md#python-helpers) helper class\.   

```
index_key = {
        'partition_key': 'key1',
        'sort_key': 55
}

encryption_context = EncryptionContext(
    table_name=table_name,
    partition_key_name=table_info.primary_index.partition,
    sort_key_name=table_info.primary_index.sort,
    # The Direct KMS Provider uses only the primary index attributes.
    # These attributes need to be formatted as a DynamoDB JSON structure.
    attributes=dict_to_ddb(index_key)
)
```

Step 5: Create the attribute actions object  
[Attribute actions](concepts.md#attribute-actions) tell the item encryptor which actions to perform on each attribute of the item\. The `AttributeActions` object in this example encrypts and signs all items except for the primary key attributes, which are signed, but not encrypted, and the `test` attribute, which is ignored\.  
When you interact directly with the item encryptor and your default action is `ENCRYPT_AND_SIGN`, you must specify an alternative action for the primary key\. You can use the `set_index_keys` method, which uses `SIGN_ONLY` for the primary key, or it uses `DO_NOTHING` if it is default action\.  
To specify the primary key, this example uses the index keys in the [TableInfo](python-using.md#python-helpers) object, which is populated by a call to DynamoDB\. This technique is safer than hard\-coding primary key names\.  

```
actions = AttributeActions(
    default_action=CryptoAction.ENCRYPT_AND_SIGN,
    attribute_actions={'test': CryptoAction.DO_NOTHING}
)
actions.set_index_keys(*table_info.protected_index_keys())
```

Step 6: Create the configuration for the item  
To configure the DynamoDB Encryption Client, use the objects that you just created in a [CryptoConfig](https://github.com/awslabs/aws-dynamodb-encryption-python/blob/master/src/dynamodb_encryption_sdk/encrypted/__init__.py) configuration for the table item\. The client helper classes create the CryptoConfig for you\.   

```
 crypto_config = CryptoConfig(
        materials_provider=aws_kms_cmp,
        encryption_context=encryption_context,
        attribute_actions=actions
    )
```

Step 7: Encrypt the item  
This step encrypts and signs the item, but it does not put it in the DynamoDB table\.   
When you use a client helper class, your items are transparently encrypted and signed, and then added to your DynamoDB table when you call the `put_item` method of the helper class\. When you use the item encryptor directly, the encrypt and put actions are separate\. This lets you customize your actions and use the DynamoDB Encryption Client on structured data that is not saved in DynamoDB\.  
The example uses the `encrypt_python_item` function, which converts the `plaintext_item` dictionary to a DynamoDB item, encrypts and signs it, and converts it back into a dictionary\. It requires a `CryptoConfig` configuration object\.   

```
plaintext_item = {
    'partition_key': 'key1',
    'sort_key': 55,
    'example': 'data',
    'some numbers': 99,
    'some binary': Binary(b'\x00\x01\x02'),
    'test': 'testdata'
}

encrypted_item = encrypt_python_item(plaintext_item, crypto_config)
```

Step 8: Put the item in the table  
This step puts the encrypted and signed item in the DynamoDB table\.  

```
table.put_item(Item=encrypted_item)
```

## Use the Direct KMS Provider<a name="python-example-streams"></a>

The following example shows you how to use the [Direct KMS Provider](direct-kms-provider.md) with the [EncryptedTable](https://github.com/awslabs/aws-dynamodb-encryption-python/blob/master/examples/src/aws_kms_encrypted_table.py) client helper class\. 

## Use the Wrapped Materials Provider<a name="python-example-multiple-providers"></a>

The following example shows you how to use the AWS Encryption SDK with more than one master key provider\. Using more than one master key provider creates redundancy if one master key provider is unavailable for decryption\. This example uses a CMK in [AWS KMS](https://aws.amazon.com/kms/) and an RSA key pair as the master keys\.

## Use the Most Recent Provider<a name="python-example-mrp"></a>

The following example shows you how to use the AWS Encryption SDK to encrypt and decrypt byte streams\. This example doesn't use AWS\. It uses a static, ephemeral master key provider\.