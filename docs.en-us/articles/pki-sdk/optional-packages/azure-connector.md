﻿# Azure Connector

The package [Lacuna PKI Azure Connector](https://www.nuget.org/packages/Lacuna.Pki.AzureConnector/) enables the
following integrations between the PKI SDK and the Microsoft Azure storage service:

* Compress and decompress CAdES signatures storing the CRLs and certificates in
  a [Blob Storage](https://docs.microsoft.com/pt-br/azure/storage/storage-dotnet-how-to-use-blobs)
* Perform and validate CAdES signatures with revocation references but without
  revocation values (CAdES-X Type 1 or ICP-Brasil AD-RV) by storing the
  correspondent values on a Blob Storage
* Send log messages generated by the SDK to a [Table Storage](https://docs.microsoft.com/pt-br/azure/storage/storage-dotnet-how-to-use-tables)

## Azure Storage credentials

The first thing you'll need is to create an Azure storage account (obviously) and get access credentials, called
"Access Keys" in Azure-vernacular. If haven't already done that, the following link can guide you through the process:

* [About Azure storage accounts](https://azure.microsoft.com/en-us/documentation/articles/storage-create-storage-account/)

By the end of that process you should have a connection string similar to:

	DefaultEndpointsProtocol=https;AccountName=youraccount;AccountKey=XXXXXXXXXX==

The best place to put that string is in an appSetting in your `web.config`:

```xml
<appSettings>
	<add key="AzureStorage" value="DefaultEndpointsProtocol=https;AccountName=youraccount;AccountKey=XXXXXXXXXX==" />
</appSettings>
```

But you may also keep it in your code, as we'll see in a bit.

## AzureBlobStorageStore

The @Lacuna.Pki.AzureConnector.AzureBlobStorageStore class implements the @Lacuna.Pki.Stores.ISimpleStore interface,
which is used by the SDK whenever a storage is needed to store and/or retrieve objects, for instance when compressing
CAdES signatures (for more information, see [Optional nuget packages](index.md)).

If you stored your Azure Storage connection string in a appSettings entry as suggested above, all you need to do to
instantiate an `AzureBlobStorageStore` is:

```cs
var store = AzureBlobStorageStore.CreateFromSettingName("AzureStorage"); // or whatever else you put in the "key" attribute of the appSetting
``` 

If you'd rather have the connection string in your code, you can:

```cs
var store = AzureBlobStorageStore.CreateFromConnectionString("DefaultEndpointsProtocol=https;AccountName=youraccount;AccountKey=XXXXXXXXXX==");
```

By default, the objects are stored in a storage named "lacuna-pki-store", but you can change that:

```cs
var store = AzureBlobStorageStore.CreateFromSettingName("AzureStorage", "my-container");
```

Once you have an instance of `AzureBlobStorageStore` associated with your storage account and container, you can,
for instance, compress and decompress a CAdES signature:

```cs
byte[] precomputedSignature = ...; // any CAdES signature, not necessarily generated with the SDK
var compressedSignature = CadesSignatureCompression.Compress(precomputedSignature, store);
var decompressedSignature = CadesSignatureCompression.Decompress(compressedSignature, store);
// precomputedSignature and decompressedSignature will be the same
```

## AzureTableStorageLogger

The @Lacuna.Pki.AzureConnector.AzureTableStorageLogger class is used to send log messages generated by the SDK to an
Azure Table. You instantiate the class in the same manner as the `AzureBlobStorageStore`:

```cs
var logger = AzureTableStorageLogger.CreateFromSettingName("AzureStorage"); // or whatever else you put in the "key" attribute of the appSetting
```

By default, logs are stored in a table named "LacunaPkiLog", but you can override that setting:

```cs
var logger = AzureTableStorageLogger.CreateFromSettingName("AzureStorage", "MyTable");
```

You can also specify the minimum level to log (the default is "Info"):

```cs
var logger = AzureTableStorageLogger.CreateFromSettingName("AzureStorage", minLevel: LogLevels.Trace); // this would log A LOT, use only for diagnostics
```

Once you have an instance of `AzureTableStorageLogger`, simply call the
@Lacuna.Pki.AzureConnector.AzureTableStorageLogger.Configure method and you're good to go:

```cs
logger.Configure();
```

In short, you generally do:

```cs
AzureTableStorageLogger.CreateFromSettingName("AzureStorage").Configure();
```

## Dependency on the package Azure Storage

In order to maximise compatibility, the package depends on an old version of the Azure Storage package, but we strongly
recommend that you install the latest version.

## Source code

The package is open source, sourced on [BitBucket](https://bitbucket.org/Lacunas/pkiazureconnector). Feel free to fork
it if you need to make any customizations.
