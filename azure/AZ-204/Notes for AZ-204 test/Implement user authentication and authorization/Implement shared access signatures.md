- Identify the three types of shared access signatures
- Explain when to implement shared access signatures
- Create a stored access policy

# Discover shared access signatures
A shared access signature (SAS) is a signed URI that points to one or more storage resources and includes a token that contains a special set of query parameters. This signature is used by Azure Storage to authorize access to the storage resource.


## Types of shared access signatures

User delegation SAS: A user delegation SAS is secured with Azure Active Directory credentials and also by the permissions specified for the SAS. A user delegation SAS applies to Blob storage only.

Service SAS: A service SAS is secured with the storage account key. A service SAS delegates access to a resource in the following Azure Storage services: Blob storage, Queue storage, Table storage, or Azure Files.

Account SAS: An account SAS is secured with the storage account key. An account SAS delegates access to resources in one or more of the storage services. All of the operations available via a service or user delegation SAS are also available via an account SAS.

**Microsoft recommends that you use Azure Active Directory credentials when possible as a security best practice, rather than using the account key, which can be more easily compromised. When your application design requires shared access signatures for access to Blob storage, use Azure Active Directory credentials to create a user delegation SAS when possible for superior security**

## How shared access signatures work

you need two components. The first is a URI to the resource you want to access. 
The second part is a SAS token that you've created to authorize access to that resource.

URI : https://medicalrecords.blob.core.windows.net/patient-images/patient-116139-nq8z7f.jpg?
SAS token: `sp=r&st=2020-01-20T11:42:32Z&se=2020-01-20T19:42:32Z&spr=https&sv=2019-02-02&sr=b&sig=SrW1HZ5Nb6MbRzTbXCaPm%2BJiSEn15tC91Y4umMPwVZs%3D`

The SAS token itself is made up of several components:

`sp=r`	Controls the access rights. The values can be a for add, c for create, d for delete, l for list, r for read, or w for write. This example is read only. The example sp=acdlrw grants all the available rig

`st=2020-01-20T11:42:32Z`	The date and time when access starts.

`se=2020-01-20T19:42:32Z`	The date and time when access ends. This example grants eight hours of access.

`sv=2019-02-02`	The version of the storage API to use.

`sr=b`	The kind of storage being accessed. In this example, b is for blob.

## Best practices

- To securely distribute a SAS and prevent man-in-the-middle attacks, always use HTTPS.
- The most secure SAS is a user delegation SAS. Use it wherever possible because it removes the need to store your storage account key in code. You must use Azure Active Directory to manage credentials. This option might not be possible for your solution.
- Try to set your expiration time to the smallest useful value. If a SAS key becomes compromised, it can be exploited for only a short time.
- Apply the rule of minimum-required privileges. Only grant the access that's required. For example, in your app, read-only access is sufficient.
- There are some situations where a SAS isn't the correct solution. When there's an unacceptable risk of using a SAS, create a middle-tier service to manage users and their access to storage.

# Choose when to use shared access signatures
Use a SAS when you want to provide secure access to resources in your storage account to any client who does not otherwise have permissions to those resources.

A common scenario where a SAS is useful is a service where users read and write their own data to your storage account. In a scenario where a storage account stores user data, there are two typical design patterns:

Clients upload and download data via a front-end proxy service, which performs authentication. This front-end proxy service has the advantage of allowing validation of business rules, but for large amounts of data or high-volume transactions, creating a service that can scale to match demand may be expensive or difficult.

A lightweight service authenticates the client as needed and then generates a SAS. Once the client application receives the SAS, they can access storage account resources directly with the permissions defined by the SAS and for the interval allowed by the SAS. The SAS mitigates the need for routing all data through the front-end proxy service.


Additionally, a SAS is required to authorize access to the source object in a copy operation in certain scenarios:

- When you copy a blob to another blob that resides in a different storage account, you must use a SAS to authorize access to the source blob. You can optionally use a SAS to authorize access to the destination blob as well.
- 
- When you copy a file to another file that resides in a different storage account, you must use a SAS to authorize access to the source file. You can optionally use a SAS to authorize access to the destination file as well.
- 
- When you copy a blob to a file, or a file to a blob, you must use a SAS to authorize access to the source object, even if the source and destination objects reside within the same storage account.

# Explore stored access policies
The following storage resources support stored access policies:
- Blob containers
- File shares
- Queues
- Tables

## Creating a stored access policy
The access policy for a SAS consists of the start time, expiry time, and permissions for the signature.  However, you cannot specify a given parameter on both the SAS token and the stored access policy.

To create or modify a stored access policy, call the `Set ACL` operation for the resource (https://docs.microsoft.com/en-us/rest/api/storageservices/set-container-acl)

**When you establish a stored access policy on a container, table, queue, or share, it may take up to 30 seconds to take effect. During this time requests against a SAS associated with the stored access policy may fail with status code 403 (Forbidden), until the access policy becomes active. Table entity range restrictions (startpk, startrk, endpk, and endrk) cannot be specified in a stored access policy.**

```
BlobSignedIdentifier identifier = new BlobSignedIdentifier
{
    Id = "stored access policy identifier",
    AccessPolicy = new BlobAccessPolicy
    {
        ExpiresOn = DateTimeOffset.UtcNow.AddHours(1),
        Permissions = "rw"
    }
};

blobContainer.SetAccessPolicy(permissions: new BlobSignedIdentifier[] { identifier });
```

```
az storage container policy create \
    --name <stored access policy identifier> \
    --container-name <container name> \
    --start <start time UTC datetime> \
    --expiry <expiry time UTC datetime> \
    --permissions <(a)dd, (c)reate, (d)elete, (l)ist, (r)ead, or (w)rite> \
    --account-key <storage account key> \
    --account-name <storage account name> \ 
```

## Modifying or revoking a stored access policy
To modify the parameters of the stored access policy you can call the access control list operation for the resource type to replace the existing policy.

To revoke a stored access policy you can delete it, rename it by changing the signed identifier, or change the expiry time to a value in the past.

To remove a single access policy, call the resource's Set ACL operation, passing in the set of signed identifiers that you wish to maintain on the container. 

To remove all access policies from the resource, call the Set ACL operation with an empty request body.

## Knowledge check

1. Which of the following types of shared access signatures (SAS) applies to Blob storage only? 
User delegation SAS

Which of the following best practices provides the most flexible and secure way to use a service or account shared access signature (SAS)?

Associate SAS tokens with a stored access policy.


