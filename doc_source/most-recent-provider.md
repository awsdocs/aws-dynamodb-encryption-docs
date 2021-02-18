# Most Recent Provider<a name="most-recent-provider"></a>

The *Most Recent Provider* is a [cryptographic materials provider](concepts.md#concept-material-provider) \(CMP\) that is designed to work with a [provider store](concepts.md#provider-store)\. It gets CMPs from the provider store, and gets the cryptographic materials that it returns from the CMPs\. It typically uses each CMP to satisfy multiple requests for cryptographic materials\. But you can use the features of its provider store to control the extent to which materials are reused, determine how often its CMP is rotated, and even change the type of CMP that it uses without changing the Most Recent Provider\.

**Note**  
The code associated with the `MostRecentProvider` symbol for the Most Recent Provider might store cryptographic materials in memory for the lifetime of the process\. It might allow a caller to use keys that they're no longer authorized to use\.   
The `MostRecentProvider` symbol is deprecated in older supported versions of the DynamoDB Encryption Client and removed from version 2\.0\.0\. It is replaced by the `CachingMostRecentProvider` symbol\. For details, see [Updates to the Most Recent Provider](#mrp-versions)\.

The Most Recent Provider is a good choice for applications that need to minimize calls to the provider store and its cryptographic source, and applications that can reuse some cryptographic materials without violating their security requirements\. For example, It allows you to protect your cryptographic materials under an [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) [customer master key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) without calling AWS KMS every time you encrypt or decrypt an item\.

The provider store that you choose determines the type of CMPs that the Most Recent Provider uses and how often it gets a new CMP\. You can use any compatible provider store with the Most Recent Provider, including custom provider stores that you design\. 

The DynamoDB Encryption Client includes a *MetaStore* that creates and returns [Wrapped Materials Providers](wrapped-provider.md) \(Wrapped CMPs\)\. The MetaStore saves multiple versions of the Wrapped CMPs that it generates in an internal DynamoDB table and protects them with client\-side encryption by an internal instance of the DynamoDB Encryption Client\. 

You can configure the MetaStore to use any type of internal CMP to protect the materials in the table, including a [Direct KMS Provider](direct-kms-provider.md) that generates cryptographic materials protected by your AWS KMS customer master key, a Wrapped CMP that uses wrapping and signing keys that you supply, or a compatible custom CMP that you design\.

**For example code, see:**
+ Java: [MostRecentEncryptedItem](https://github.com/aws/aws-dynamodb-encryption-java/blob/master/examples/src/main/java/com/amazonaws/examples/MostRecentEncryptedItem.java)
+ Python: [most\_recent\_provider\_encrypted\_table](https://github.com/aws/aws-dynamodb-encryption-python/blob/master/examples/src/most_recent_provider_encrypted_table.py)

**Topics**
+ [How to use it](#mrp-how-to-use-it)
+ [How it works](#mrp-how-it-works)
+ [Updates to the Most Recent Provider](#mrp-versions)

## How to use it<a name="mrp-how-to-use-it"></a>

To create a Most Recent Provider, you need to create and configure a provider store, and then create a Most Recent Provider that uses the provider store\. 

The following examples show how to create a Most Recent Provider that uses a MetaStore and protects the versions in its internal DynamoDB table with cryptographic materials from a [Direct KMS Provider](direct-kms-provider.md)\. These examples use the [`CachingMostRecentProvider`](#mrp-versions) symbol\. 

Each Most Recent Provider has a name that identifies its CMPs in the MetaStore table, a [time\-to\-live](#most-recent-provider-ttl) \(TTL\) setting, and a cache size setting that determines how many entries the cache can hold\. These examples set the cache size to 1000 entries and a TTL of 60 seconds\.

------
#### [ Java ]

```
// Set the name for MetaStore's internal table
final String keyTableName = 'metaStoreTable'

// Set the Region and CMK for AWS KMS
final String region = 'us-west-2'
final String cmkArn = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

// Set the TTL and cache size
final long ttlInMillis = 60000;
final long cacheSize = 1000;

// Name that identifies the MetaStore's CMPs in the provider store
final String materialName = 'testMRP'

// Create an internal DynamoDB client for the MetaStore
final AmazonDynamoDB ddb = AmazonDynamoDBClientBuilder.standard().withRegion(region).build();

// Create an internal Direct KMS Provider for the MetaStore
final AWSKMS kms = AWSKMSClientBuilder.standard().withRegion(region).build();
final DirectKmsMaterialProvider kmsProv = new DirectKmsMaterialProvider(kms, cmkArn);

// Create an item encryptor for the MetaStore,
// including the Direct KMS Provider
final DynamoDBEncryptor keyEncryptor = DynamoDBEncryptor.getInstance(kmsProv);

// Create the MetaStore
final MetaStore metaStore = new MetaStore(ddb, keyTableName, keyEncryptor);

//Create the Most Recent Provider
final CachingMostRecentProvider cmp = new CachingMostRecentProvider(metaStore, materialName, ttlInMillis, cacheSize);
```

------
#### [ Python ]

```
# Designate an AWS KMS customer master key
aws_cmk_id = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

# Set the name for MetaStore's internal table
meta_table_name = 'metaStoreTable'

# Name that identifies the MetaStore's CMPs in the provider store
material_name = 'testMRP'

# Create an internal DynamoDB table resource for the MetaStore
meta_table = boto3.resource('dynamodb').Table(meta_table_name)

# Create an internal Direct KMS Provider for the MetaStore
aws_kms_cmp = AwsKmsCryptographicMaterialsProvider(key_id=aws_cmk_id)
    
# Create the MetaStore with the Direct KMS Provider
meta_store = MetaStore(
    table=meta_table,
    materials_provider=aws_kms_cmp
)

# Create a Most Recent Provider using the MetaStore
#    Sets the TTL (in seconds) and cache size (# entries)
most_recent_cmp = MostRecentProvider(
    provider_store=meta_store,
    material_name=material_name,
    version_ttl=60.0,
    cache_size=1000
)
```

------

## How it works<a name="mrp-how-it-works"></a>

The Most Recent Provider gets CMPs from a provider store\. Then, it uses the CMP to generate the cryptographic materials that it returns to the item encryptor\.

### About the Most Recent Provider<a name="about-mrp"></a>

The Most Recent Provider gets a [cryptographic materials provider](concepts.md#concept-material-provider) \(CMP\) from a [provider store](concepts.md#provider-store)\. Then, it uses the CMP to generate the cryptographic materials it returns\. Each Most Recent Provider is associated with one provider store, but a provider store can supply CMPs to multiple providers across multiple hosts\.

The Most Recent Provider can work with any compatible CMP from any provider store\. It requests encryption or decryption materials from the CMP and returns the output to the item encryptor\. It does not perform any cryptographic operations\.

To request a CMP from its provider store, the Most Recent Provider supplies its material name and the version of an existing CMP it wants to use\. For encryption materials, the Most Recent Provider always requests the maximum \("most recent"\) version\. For decryption materials, it requests the version of the CMP that was used to create the encryption materials, as shown in the following diagram\.

![\[A Most Recent Provider\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/most-recent-provider-1.png)

The Most Recent Provider saves versions of the CMPs that the provider store returns in a local Least Recently Used \(LRU\) cache in memory\. The cache enables the Most Recent Provider to get the CMPs that it needs without calling the provider store for every item\. You can clear the cache on demand\.

The Most Recent Provider uses a configurable [time\-to\-live value](#most-recent-provider-ttl) that you can adjust based on the characteristics of your application\.

### About the MetaStore<a name="about-metastore"></a>

You can use a Most Recent Provider with any provider store, including a compatible custom provider store\. The DynamoDB Encryption Client includes a MetaStore, a secure implementation that you can configure and customize\.

A *MetaStore* is a [provider store](concepts.md#provider-store) that creates and returns [Wrapped CMPs](wrapped-provider.md) that are configured with the wrapping key, unwrapping key, and signing key that Wrapped CMPs require\. A MetaStore is a secure option for a Most Recent Provider because Wrapped CMPs always generate unique item encryption keys for every item\. Only the wrapping key that protects the item encryption key and the signing keys are reused\.

The following diagram shows the components of the MetaStore and how it interacts with the Most Recent Provider\.

![\[A MetaStore\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/most-recent-provider-2.png)

The MetaStore generates the Wrapped CMPs, and then stores them \(in encrypted form\) in an internal DynamoDB table\. The partition key is the name of the Most Recent Provider material; the sort key its version number\. The materials in the table are protected by an internal DynamoDB Encryption Client, including an item encryptor and internal [cryptographic materials provider](concepts.md#concept-material-provider) \(CMP\)\.

You can use any type of internal CMP in your MetaStore, including the a [Direct KMS Provider](wrapped-provider.md), a Wrapped CMP with cryptographic materials that you provide, or a compatible custom CMP\. If the internal CMP in your MetaStore is a Direct KMS Provider, your reusable wrapping and signing keys are protected under your [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) [customer master key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys)\. The MetaStore calls AWS KMS every time it adds a new CMP version to its internal table or gets a CMP version from its internal table\.

### Setting a time\-to\-live value<a name="most-recent-provider-ttl"></a>

You can set a time\-to\-live \(TTL\) value for each Most Recent Provider that you create\. In general, use the lowest TTL value that is practical for your application\.

The use of the TTL value is changed in the `CachingMostRecentProvider` symbol for the Most Recent Provider\. 

**Note**  
The `MostRecentProvider` symbol for the Most Recent Provider is deprecated in older supported versions of the DynamoDB Encryption Client and removed from version 2\.0\.0\. It is replaced by the `CachingMostRecentProvider` symbol\. We recommend that you update your code as soon as possible\. For details, see [Updates to the Most Recent Provider](#mrp-versions)\.

**`CachingMostRecentProvider`**  
The `CachingMostRecentProvider` uses the TTL value in two different ways\.   
+ The TTL determines how often the Most Recent Provider checks the provider store for a new version of the CMP\. If a new version is available, the Most Recent Provider replaces its CMP and refreshes its cryptographic materials\. Otherwise, it continues to use its current CMP and cryptographic materials\.
+ The TTL determines how long CMPs in the cache can be used\. Before it uses a cached CMP for encryption, the Most Recent Provider evaluates its time in the cache\. If the CMP cache time exceeds the TTL, the CMP is evicted from the cache and the Most Recent Provider gets a new, latest\-version CMP from its provider store\.

**`MostRecentProvider`**  
In the `MostRecentProvider`, the TTL determines how often the Most Recent Provider checks the provider store for a new version of the CMP\. If a new version is available, the Most Recent Provider replaces its CMP and refreshes its cryptographic materials\. Otherwise, it continues to use its current CMP and cryptographic materials\.

The TTL does not determine how often a new CMP version is created\. You create new CMP versions by [rotating the cryptographic materials](#most-recent-provider-rotate)\.

An ideal TTL value varies with the application and its latency and availability goals\. A lower TTL improves your security profile by reducing the time that cryptographic materials are stored in memory\. Also, a lower TTL refreshes critical information more frequently\. For example, if your internal CMP is a [Direct KMS Provider](direct-kms-provider.md), it verifies more frequently that the caller is still authorized to use an AWS KMS customer master key\. 

However, if the TTL is too brief, the frequent calls to the provider store can increase your costs and cause your provider store to throttle requests from your application and other applications that share your service account\. You might also benefit from coordinating the TTL with the rate at which you rotate cryptographic materials\. 

During testing, vary the TTL and cache size under different work loads until you find a configuration that works for your application and your security and performance standards\.

### Rotating cryptographic materials<a name="most-recent-provider-rotate"></a>

When a Most Recent Provider needs encryption materials, it always uses the most recent version of its CMP that it knows about\. The frequency that it checks for a newer version is determined by the [time\-to\-live](#most-recent-provider-ttl) \(TTL\) value that you set when you configure the Most Recent Provider\. 

When the TTL expires, the Most Recent Provider checks the provider store for newer version of the CMP\. If one is available, the Most Recent Provider get it and replaces the CMP in its cache\. It uses this CMP and its cryptographic materials until it discovers that provider store has a newer version\.

To tell the provider store to create a new version of a CMP for a Most Recent Provider, call the provider store's Create New Provider operation with the material name of the Most Recent Provider\. The provider store creates a new CMP and saves an encrypted copy in its internal storage with a greater version number\. \(It also returns a CMP, but you can discard it\.\) As a result, the next time the Most Recent Provider queries the provider store for the maximum version number of its CMPs, it gets the new greater version number, and uses it in subsequent requests to the store to see if a new version of the CMP has been created\.

You can schedule your Create New Provider calls based on time, the number of items or attributes processed, or any other metric that makes sense for your application\.

### Get encryption materials<a name="most-recent-provider-encrypt"></a>

The Most Recent Provider uses the following process, shown in this diagram, to get the encryption materials that it returns to the item encryptor\. The output depends on the type of CMP that the provider store returns\. The Most Recent Provider can use any compatible provider store, including the MetaStore that is included in the DynamoDB Encryption Client\.

![\[Input, processing, and output of the Most Recent Provider in the DynamoDB Encryption Client\]](http://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/images/most-recent-provider-provider-store.png)

When you create a Most Recent Provider by using the [`CachingMostRecentProvider` symbol](#mrp-versions), you specify a provider store, a name for the Most Recent Provider, and a [time\-to\-live](#most-recent-provider-ttl) \(TTL\) value\. You can also optionally specify a cache size, which determines the maximum number of cryptographic materials that can exist in the cache\.

When the item encryptor asks the Most Recent Provider for encryption materials, the Most Recent Provider begins by searching its cache for the latest version of its CMP\.
+ If it finds the latest version CMP in its cache and the CMP has not exceeded the TTL value, the Most Recent Provider uses the CMP to generate encryption materials\. Then, it returns the encryption materials to the item encryptor\. This operation does not require a call to the provider store\.
+ If the latest version of the CMP is not in its cache, or if it is in the cache but has exceeded its TTL value, the Most Recent Provider requests a CMP from its provider store\. The request includes the Most Recent Provider material name and the maximum version number that it knows\.

  1. The provider store returns a CMP from its persistent storage\. If the provider store is a MetaStore, it gets an encrypted Wrapped CMP from its internal DynamoDB table by using the Most Recent Provider material name as the partition key and the version number as the sort key\. The MetaStore uses its internal item encryptor and internal CMP to decrypt the Wrapped CMP\. Then, it returns the plaintext CMP to the Most Recent Provider \. If the internal CMP is a [Direct KMS Provider](direct-kms-provider.md), this step includes a call to the [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\)\.

  1. The CMP adds the `amzn-ddb-meta-id` field to the [actual material description](concepts.md#material-description)\. Its value is the material name and version of the CMP in its internal table\. The provider store returns the CMP to the Most Recent Provider\.

  1. The Most Recent Provider caches the CMP in memory\.

  1. The Most Recent Provider uses the CMP to generate encryption materials\. Then, it returns the encryption materials to the item encryptor\.

### Get decryption materials<a name="most-recent-provider-decrypt"></a>

When the item encryptor asks the Most Recent Provider for decryption materials, the Most Recent Provider uses the following process to get and return them\.

1. The Most Recent Provider asks the provider store for the version number of the cryptographic materials that were used to encrypt the item\. It passes in the actual material description from the [material description attribute](concepts.md#material-description) of the item\.

1. The provider store gets the encrypting CMP version number from the `amzn-ddb-meta-id` field in the actual material description and returns it to the Most Recent Provider\.

1. The Most Recent Provider searches its cache for the version of CMP that was used to encrypt and sign the item\.
+ If it finds the matching version of the CMP is in its cache and the CMP has not exceeded the [time\-to\-live \(TTL\) value](#most-recent-provider-ttl), the Most Recent Provider uses the CMP to generate decryption materials\. Then, it returns the decryption materials to the item encryptor\. This operation does not require a call to the provider store or any other CMP\.
+ If the matching version of the CMP is not in its cache, or if the cached CMK has exceeded its TTL value, the Most Recent Provider requests a CMP from its provider store\. It sends its material name and the encrypting CMP version number in the request\.

  1. The provider store searches its persistent storage for the CMP by using the Most Recent Provider name as the partition key and the version number as the sort key\.
     + If the name and version number are not in its persistent storage, the provider store throws an exception\. If the provider store was used to generate the CMP, the CMP should be stored in its persistent storage, unless it was intentionally deleted\.
     + If the CMP with the matching name and version number are in the provider store's persistent storage, the provider store returns the specified CMP to the Most Recent Provider\. 

       If the provider store is a MetaStore, it gets the encrypted CMP from its DynamoDB table\. Then, it uses cryptographic materials from its internal CMP to decrypt the encrypted CMP before it returns the CMP to Most Recent Provider\. If the internal CMP is a [Direct KMS Provider](direct-kms-provider.md), this step includes a call to the [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\)\.

  1. The Most Recent Provider caches the CMP in memory\.

  1. The Most Recent Provider uses the CMP to generate decryption materials\. Then, it returns the decryption materials to the item encryptor\.

## Updates to the Most Recent Provider<a name="mrp-versions"></a>

The symbol for the Most Recent Provider is changed from `MostRecentProvider` to `CachingMostRecentProvider`\. 

**Note**  
The `MostRecentProvider` symbol, which represents the Most Recent Provider, is deprecated in version 1\.15 of the DynamoDB Encryption Client for Java and version 1\.3 of the DynamoDB Encryption Client for Python and removed from versions 2\.0\.0 of the DynamoDB Encryption Client in both language implementations\. Instead, use the `CachingMostRecentProvider`\.

The `CachingMostRecentProvider` implements the following changes:
+ The `CachingMostRecentProvider` periodically removes cryptographic materials from memory when their time in memory exceeds the configured [time\-to\-live \(TTL\) value](#most-recent-provider-ttl)\. 

  The `MostRecentProvider` might store cryptographic materials in memory for the lifetime of the process\. As a result, the Most Recent Provider might not be aware of authorization changes\. It might use encryption keys after the caller's permissions to use them are revoked\. 

  If you can't update to this new version, you can get a similar effect by periodically calling the `clear()` method on the cache\. This method manually flushes the cache contents and requires the Most Recent Provider to request a new CMP and new cryptographic materials\. 
+ The `CachingMostRecentProvider` also includes a cache size setting that gives you more control over the cache\.

To update to the `CachingMostRecentProvider`, you have to change the symbol name in your code\. In all other respects, the `CachingMostRecentProvider` is fully backwards compatible with the `MostRecentProvider`\. You don't need to re\-encrypt any table items\.

However, the `CachingMostRecentProvider` generates more calls to the underlying key infrastructure\. It calls the provider store at least once in each time\-to\-live \(TTL\) interval\. Applications with numerous active CMPs \(due to frequent rotation\) or applications with large fleets are most likely to be sensitive to this change\. 

Before releasing your updated code, test it thoroughly to ensure that the more frequent calls don't impair your application or cause throttling by services on which your provider depends, such as AWS Key Management Service \(AWS KMS\) or Amazon DynamoDB\. To mitigate any performance problems, adjust the cache size and the time\-to\-live of the `CachingMostRecentProvider` based on the performance characteristics you observe\. For guidance, see [Setting a time\-to\-live value](#most-recent-provider-ttl)\.