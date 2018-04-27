# Which Fields are Encrypted and Signed?<a name="encrypted-and-signed"></a>

The DynamoDB Encryption Client encrypts the values of selected attributes and then calculates a signature over the attributes that you specify\. 

In an encrypted item, some data remains in plain text, including the table name, all attribute names, the attribute values that you do not encrypt, and the names and values of the primary key \(the partition key and sort key\) attributes\. Do not store sensitive data in these fields\.

**Encrypting Attribute Values**

The DynamoDB Encryption Client encrypts the values \(but not the names\) of the attributes that you specify\. To determine which attribute values are encrypted, use an [attribute actions](concepts.md#attribute-actions) object\. 

For example, this item includes example and test attributes\.

```
'example': 'data',
'test': 'test-value',
...
```

If you encrypt the `example` attribute, but do nothing to the `test` attribute , the results look like this\.

```
'example': Binary(b"'b\x933\x9a+s\xf1\xd6a\xc5\xd5\x1aZ\xed\xd6\xce\xe9X\xf0T\xcb\x9fY\x9f\xf3\xc9C\x83\r\xbb\\"),
'test': 'test-value'
```

Do not encrypt the values of the primary key \(partition key and sort key\) attributes\. They must remain in plain text so DynamoDB can access the item for you\. If you use a helper class, it will throw an exception if you try to encrypt the primary key\. If you need to encrypt it for a special use case, use the [item encryptor](concepts.md#item-encryptor) directly\.

The DynamoDB Encryption Client does not encrypt the [Material Description](concepts.md#material-description) attribute, which stores the data that DynamoDB Encryption Client needs to verify and decrypt the item\. 

**Signing the Item**

After encrypting the specified attribute values, the DynamoDB Encryption Client calculates a digital signature over the names and values of attributes that you specify in the [attribute actions](concepts.md#attribute-actions) object\. The signature allows you to detect unauthorized changes to the item as a whole, including adding or deleting attributes, or substituting one encrypted value for another\. The signature is saved in an attribute that the client adds to the item\.

You can \(and should\) include the primary key in the signature\. This is the default if you use a helper class, but you can override it by using the [item encryptor](concepts.md#item-encryptor) directly\. You can also include the table name in the signature\. The [Material Description](concepts.md#material-description) attribute is neither encrypted nor signed, as shown in the following image\.

![\[An encrypted and signed table item\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/encrypted-signed-item.png)