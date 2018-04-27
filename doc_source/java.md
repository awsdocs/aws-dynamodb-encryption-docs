# Amazon DynamoDB Encryption Client for Java<a name="java"></a>

This topic explains how to install and use the Amazon DynamoDB Encryption Client for Java\. For details about programming with the DynamoDB Encryption Client, see the [Java examples](java-examples.md), the [examples](https://github.com/awslabs/aws-dynamodb-encryption-java/tree/master/examples) in the aws\-dynamodb\-encryption\-java repository on GitHub and the [Javadoc](https://awslabs.github.io/aws-dynamodb-encryption-java/javadoc/) for the DynamoDB Encryption Client\.

**Topics**
+ [Prerequisites](#java-prerequisites)
+ [Installation](#java-installation)
+ [Using the DynamoDB Encryption Client for Java](java-using.md)
+ [Java Examples](java-examples.md)

## Prerequisites<a name="java-prerequisites"></a>

Before you install the Amazon DynamoDB Encryption Client for Java, be sure you have the following prerequisites\.

**A Java development environment**  
You will need Java 8 or later\. On the Oracle website, go to [Java SE Downloads](https://www.oracle.com/technetwork/java/javase/downloads/index.html), and then download and install the Java SE Development Kit \(JDK\)\.  
If you use the Oracle JDK, you must also download and install the [Java Cryptography Extension \(JCE\) Unlimited Strength Jurisdiction Policy Files](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)\.

**Bouncy Castle**  
Bouncy Castle provides a cryptography API for Java\. If you don't have Bouncy Castle, go to [Bouncy Castle latest releases](https://bouncycastle.org/latest_releases.html) to download the provider file that corresponds to your JDK\.  
If you use [Apache Maven](https://maven.apache.org/), Bouncy Castle is available with the following dependency definition\. Replace the version number in the `version` element with the one that you selected\.  

```
<dependency>
  <groupId>org.bouncycastle</groupId>
  <artifactId>bcprov-ext-jdk15on</artifactId>
  <version>version-number</version>
</dependency>
```

**AWS SDK for Java \(Optional\)**  
You do not need the AWS SDK for Java to use the Amazon DynamoDB Encryption Client for Java, but you do need it to use [AWS Key Management Service \(AWS KMS\)](https://aws.amazon.com/kms/) as a cryptographic material provider, and to use some of the example Java code in this guide\. For more information about installing and configuring the AWS SDK for Java, see [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/)\.

## Installation<a name="java-installation"></a>

You can install the Amazon DynamoDB Encryption Client for Java in the following ways\.

**Manually**  
To install the Amazon DynamoDB Encryption Client for Java, clone or download the [aws\-dynamodb\-encryption\-java](https://github.com/awslabs/aws-dynamodb-encryption-java/) GitHub repository\.

**Using Apache Maven**  
The Amazon DynamoDB Encryption Client for Java is available through [Apache Maven](https://maven.apache.org/) with the following dependency definition\.  

```
<dependency>
  <groupId>com.amazonaws</groupId>
  <artifactId>aws-dynamodb-encryption-java</artifactId>
  <version>version-number</version>
</dependency>
```

After you install the SDK, get started by looking at the example code in this guide and the [DynamoDB Encryption Client Javadoc](https://awslabs.github.io/aws-dynamodb-encryption-java/javadoc/) on GitHub\.