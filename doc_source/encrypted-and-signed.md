# Which Fields Are Encrypted and Signed?<a name="encrypted-and-signed"></a>

The DynamoDB Encryption Client encrypts the values of selected attributes, and then calculates a signature over the attributes that you specify\. 

In an encrypted item, some data remains in plaintext, including the table name, all attribute names, the attribute values that you don't encrypt, and the names and values of the primary key \(partition key and sort key\) attributes\. Do not store sensitive data in these fields\.

**Encrypting Attribute Values**

The DynamoDB Encryption Client encrypts the values \(but not the names\) of the attributes that you specify\. To determine which attribute values are encrypted, use an [attribute actions](concepts.md#attribute-actions) object\. 

For example, this item includes example and test attributes\.

```
'example': 'data',
'test': 'test-value',
...
```

If you encrypt the `example` attribute, but do nothing to the `test` attribute, the results look like this\.

```
'example': Binary(b"'b\x933\x9a+s\xf1\xd6a\xc5\xd5\x1aZ\xed\xd6\xce\xe9X\xf0T\xcb\x9fY\x9f\xf3\xc9C\x83\r\xbb\\"),
'test': 'test-value'
...
```

**Warning**  
Do not encrypt the primary key attributes\. They must remain in plaintext so DynamoDB can find the item without running a full table scan\.

By default, the primary key—partition key and sort key—attributes are signed, but not encrypted\. If you identify your primary key, or use a helper class that identifies it for you, and then try to encrypt it, the client will throw an exception\. If you need to encrypt the primary key for a special use case, use the lower\-level [item encryptor](concepts.md#item-encryptor) directly, but remember that DynamoDB will not be able to find your item\.

The DynamoDB Encryption Client also does not encrypt the [Material Description](concepts.md#material-description) attribute, which stores the data that the DynamoDB Encryption Client needs to verify and decrypt the item\. 

**Signing the Item**

After encrypting the specified attribute values, the DynamoDB Encryption Client calculates a digital signature over the names and values of attributes that you specify in the [attribute actions](concepts.md#attribute-actions) object\. If you specify a table name, it's included in the signature, too\.

![\[An encrypted and signed table item\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/encrypted-signed-item.png)

The signature allows you to detect unauthorized changes to the item as a whole, including adding or deleting attributes, or substituting one encrypted value for another\. The signature is saved in an attribute that the client adds to the item\.

Be sure to include the primary key in the signature\. It's the default behavior when you use a helper class\. The signature captures the relationship between the primary key and other attributes in the item, and the signature validation verifies that the relationship hasn't changed\. 

The [Material Description](concepts.md#material-description) attribute is not encrypted or signed\.