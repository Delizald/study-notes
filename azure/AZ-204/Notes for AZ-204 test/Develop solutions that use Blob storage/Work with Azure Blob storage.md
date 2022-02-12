# Learning objectives

- Create an application to create and manipulate data by using the Azure Storage client library for Blob storage.
- Manage container properties and metadata by using .NET and REST.

# Explore Azure Blob storage client library

Below are the classes in the `Azure.Storage.Blobs` namespace and their purpose:

- `BlobClient`: allows you to manipulate Azure Storage blobs.
- `BlobClientOptions`: Provides the client configuration options for connecting to Azure Blob Storage.
- `BlobContainerClient`: allows you to manipulate Azure Storage containers and their blobs.
- `BlobServiceClient`: allows you to manipulate Azure Storage service resources and blob containers. The storage account provides the top-level namespace for the Blob service.
- `BlobUriBuilder`: provides a convenient way to modify the contents of a Uri instance to point to different Azure Storage resources like an account, container, or blob.


# Exercise: Create Blob storage resources by using the .NET client library
https://docs.microsoft.com/en-us/learn/modules/work-azure-blob-storage/3-develop-blob-storage-dotnet

# Manage container properties and metadata by using .NET
(https://docs.microsoft.com/en-us/learn/modules/work-azure-blob-storage/4-manage-container-properties-metadata-dotnet)

- System properties: System properties exist on each Blob storage resource. Some of them can be read or set, while others are read-only. Under the covers, some system properties correspond to certain standard HTTP headers. The Azure Storage client library for .NET maintains these properties for you.
  
- User-defined metadata: User-defined metadata consists of one or more name-value pairs that you specify for a Blob storage resource. You can use metadata to store additional values with the resource. Metadata values are for your own purposes only, and do not affect how the resource behaves.

## Retrieve container properties
- GetProperties
- GetPropertiesAsync

```
private static async Task ReadContainerPropertiesAsync(BlobContainerClient container)
{
    try
    {
        // Fetch some container properties and write out their values.
        var properties = await container.GetPropertiesAsync();
        Console.WriteLine($"Properties for container {container.Uri}");
        Console.WriteLine($"Public access level: {properties.Value.PublicAccess}");
        Console.WriteLine($"Last modified time in UTC: {properties.Value.LastModified}");
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine($"HTTP error code {e.Status}: {e.ErrorCode}");
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

## Set and retrieve metadata
To set metadata, add name-value pairs to an `IDictionary` object, and then call one of the following methods of the `BlobContainerClient` class:

- SetMetadata
- SetMetadataAsync

**The name of your metadata must conform to the naming conventions for C# identifiers.**

Metadata names preserve the case with which they were created, but are case-insensitive when set or read.

f two or more metadata headers with the same name are submitted for a resource, the Blob service returns status code 400 (Bad Request).

```
public static async Task AddContainerMetadataAsync(BlobContainerClient container)
{
    try
    {
        IDictionary<string, string> metadata =
           new Dictionary<string, string>();

        // Add some metadata to the container.
        metadata.Add("docType", "textDocuments");
        metadata.Add("category", "guidance");

        // Set the container's metadata.
        await container.SetMetadataAsync(metadata);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine($"HTTP error code {e.Status}: {e.ErrorCode}");
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

# Set and retrieve properties and metadata for blob resources by using REST
## Metadata header format


The format for the header is: `x-ms-meta-name:string-value`

The metadata consists of name/value pairs. The total size of all metadata pairs can be up to 8KB in size.

## Operations on metadata
Note that metadata values can only be read or written in full; partial updates are not supported. Setting metadata on a resource overwrites any existing metadata values for that resource.

## Retrieving properties and metadata
The GET and HEAD operations both retrieve metadata headers for the specified container or blob

`GET/HEAD https://myaccount.blob.core.windows.net/mycontainer?restype=container`

The URI syntax for retrieving metadata headers on a blob is as follows:

`GET/HEAD https://myaccount.blob.core.windows.net/mycontainer/myblob?comp=metadata`

## Setting Metadata Headers

The PUT operation sets metadata headers on the specified container or blob, overwriting any existing metadata on the resource

`PUT https://myaccount.blob.core.windows.net/mycontainer?comp=metadata&restype=container`

The URI syntax for setting metadata headers on a blob is as follows:

`PUT https://myaccount.blob.core.windows.net/mycontainer/myblob?comp=metadata`

## Standard HTTP properties for containers and blobs

Metadata headers are named with the header prefix x-ms-meta- and a custom name.

Property headers use standard HTTP header names, as specified in the Header Field Definitions section 14 of the HTTP/1.1 protocol specification.

standard HTTP headers supported on containers:

- `ETag`
- `Last-Modified`

standard HTTP headers supported on blobs:

- `ETag`
- `Last-Modified`
- `Content-Length`
- `Content-Type`
- `Content-MD5`
- `Content-Encoding`
- `Content-Language`
- `Cache-Control`
- `Origin`
- `Range`

## Knowledge check

1. Which of the following standard HTTP headers are supported for both containers and blobs when setting properties by using REST?

Last-Modified

2. Which of the following classes of the Azure Storage client library for .NET allows you to manipulate both Azure Storage containers and their blobs?

