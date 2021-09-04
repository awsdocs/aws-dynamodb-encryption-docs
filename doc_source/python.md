# DynamoDB Encryption Client for Python<a name="python"></a>

This topic explains how to install and use the DynamoDB Encryption Client for Python\. You can find the code in the [aws\-dynamodb\-encryption\-python](https://github.com/aws/aws-dynamodb-encryption-python/) repository on GitHub, including complete and tested [sample code](https://github.com/aws/aws-dynamodb-encryption-python/tree/master/examples) to help you get started\.

**Topics**
+ [Prerequisites](#python-prerequisites)
+ [Installation](#python-installation)
+ [Using the DynamoDB Encryption Client for Python](python-using.md)
+ [Python examples](python-examples.md)

## Prerequisites<a name="python-prerequisites"></a>

Before you install the Amazon DynamoDB Encryption Client for Python, be sure you have the following prerequisites\.

**A supported version of Python**  
Python 3\.5 or later is required by the Amazon DynamoDB Encryption Client for Python versions 3\.0\.*x* and later\. To download Python, see [Python downloads](https://www.python.org/downloads/)\.  
Earlier versions of the Amazon DynamoDB Encryption Client for Python support Python 2\.7 and Python 3\.4, but we recommend that you use the latest version of the DynamoDB Encryption Client\.

**The pip installation tool for Python**  
Python 3\.5 and later include **pip**, although you might want to upgrade it\. For more information about upgrading or installing pip, see [Installation](https://pip.pypa.io/en/latest/installation/) in the **pip** documentation\.

## Installation<a name="python-installation"></a>

Use **pip** to install the Amazon DynamoDB Encryption Client for Python, as shown in the following examples\.

**To install the latest version**  

```
pip install dynamodb-encryption-sdk
```

For more details about using **pip** to install and upgrade packages, see [Installing Packages](https://packaging.python.org/tutorials/installing-packages/)\.

The DynamoDB Encryption Client requires the [cryptography library](https://cryptography.io/en/latest/) on all platforms\. All versions of **pip** install and build the **cryptography** library on Windows\. **pip** 8\.1 and later installs and builds **cryptography** on Linux\. If you are using an earlier version of **pip** and your Linux environment doesn't have the tools needed to build the **cryptography** library, you need to install them\. For more information, see [Building cryptography on Linux](https://cryptography.io/en/latest/installation/#building-cryptography-on-linux)\.

You can get the latest development version of the DynamoDB Encryption Client from the [aws\-dynamodb\-encryption\-python](https://github.com/aws/aws-dynamodb-encryption-python/) repository on GitHub\.

After you install the DynamoDB Encryption Client, get started by looking at the example Python code in this guide\.