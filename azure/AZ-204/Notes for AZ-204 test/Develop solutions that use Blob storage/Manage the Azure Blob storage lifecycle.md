# Learning objectives
- Describe how each of the access tiers are optimized.
- Create and implement a lifecycle policy.
- Rehydrate blob data stored in an archive tier.

# Explore the Azure Blob storage lifecycle

## Access tiers
- `Hot` - Optimized for storing data that is accessed frequently.
- `Cool` - Optimized for storing data that is infrequently accessed and stored for at least 30 days.
- `Archive` - Optimized for storing data that is rarely accessed and stored for at least 180 days with flexible latency requirements, on the order of hours.

The following considerations apply to the different access tiers:

- access tier can be set on a blob during or after upload.
- the hot and cool access tiers can be set at the account level.
- The archive access tier can only be set at the blob level.
- Data in the cool access tier has slightly lower availability, but still has high durability.
- Data in the archive access tier is stored offline.
- The hot and cool tiers support all redundancy options
- The archive tier supports only LRS, GRS, and RA-GRS.
- Data storage limits are set at the account level and not per access tier. You can choose to use all of your limit in one tier or across all three tiers.

## Manage the data lifecycle
The lifecycle allows you to:

- Transition blobs to a cooler storage tier (hot to cool, hot to archive, or cool to archive) to optimize for performance and cost.
- Delete blobs at the end of their lifecycles.
- Define rules to be run once per day at the storage account level.
- Apply rules to containers or a subset of blobs (using prefixes as filters.

# Discover Blob storage lifecycle policies
A lifecycle management policy is a collection of rules in a JSON document.
each rule definition within a policy includes a filter set and an action set.
```
{
  "rules": [
    {
      "name": "rule1",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {...}
    },
    {
      "name": "rule2",
      "type": "Lifecycle",
      "definition": {...}
    }
  ]
}
```

(lifecycle policies.PNG)

## Rules
The following sample rule filters the account to run the actions on objects that exist inside container1 and start with foo.

- Tier blob to cool tier 30 days after last modification
- Tier blob to archive tier 90 days after last modification
- Delete blob 2,555 days (seven years) after last modification
- Delete blob snapshots 90 days after snapshot creation

```
{
  "rules": [
    {
      "name": "ruleFoo",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "container1/foo" ]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
            "delete": { "daysAfterModificationGreaterThan": 2555 }
          },
          "snapshot": {
            "delete": { "daysAfterCreationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

## Rule filters
- `blobTypes`: An array of predefined enum values. (Required)
- `prefixMatch`: An array of strings for prefixes to be match. Each rule can define up to 10 prefixes. A prefix string must start with a container name. (Not required)
- `blobIndexMatch`: 	An array of dictionary values consisting of blob index tag key and value conditions to be matched. Each rule can define up to 10 blob index tag condition. (Not required).

## Rule Actions (see:  rule actions.png)

# Implement Blob storage lifecycle policies
(https://docs.microsoft.com/en-us/learn/modules/manage-azure-blob-storage-lifecycle/4-add-policy-blob-storage)
You can add, edit, or remove a policy by using any of the following methods:

- Azure portal
- Azure PowerShell
- Azure CLI
- REST APIs

```
az storage account management-policy create \
    --account-name <storage-account> \
    --policy @policy.json \
    --resource-group <resource-group>
```

# Rehydrate blob data from the archive tier
While a blob is in the archive access tier, it's considered to be offline and can't be read or modified. 

To be able to read from an archive tier you need to reydrate the blob:
- `Copy an archived blob to an online tier`: You can rehydrate an archived blob by copying it to a new blob in the hot or cool tier with the Copy Blob or Copy Blob from URL operation. Microsoft recommends this option for most scenarios.

- `Change a blob's access tier to an online tier`: You can rehydrate an archived blob to hot or cool by changing its tier using the Set Blob Tier operation.

Rehydrating a blob from the archive tier can take several hours to complete. Microsoft recommends rehydrating larger blobs for optimal performance. 

## Rehydration priority
you can set the priority for the rehydration operation via the optional `x-ms-rehydrate-priority` (https://docs.microsoft.com/en-us/rest/api/storageservices/set-blob-tier)

- `Standard priority`: The rehydration request will be processed in the order it was received and may take up to 15 hours.
- `High priority`: The rehydration request will be prioritized over standard priority requests and may complete in under one hour for objects under 10 GB in size.

To check the rehydration priority while the rehydration operation is underway, call `Get Blob Properties` (https://docs.microsoft.com/en-us/rest/api/storageservices/get-blob-properties)

## Copy an archived blob to an online tier
Copying an archived blob to an online destination tier is supported within the same storage account only. You cannot copy an archived blob to a destination blob that is also in the archive tier
(blob copy behaviour.PNG)

## Change a blob's access tier to an online tier
The second option for rehydrating a blob from the archive tier to an online tier is to change the blob's tier by calling Set Blob Tier. 
Once a Set Blob Tier request is initiated, it cannot be canceled. During the rehydration operation, the blob's access tier setting continues to show as archived until the rehydration process is complete.

(https://docs.microsoft.com/en-us/azure/storage/blobs/archive-rehydrate-to-online-tier#rehydrate-a-blob-by-changing-its-tier)

**Changing a blob's tier doesn't affect its last modified time. If there is a lifecycle management policy in effect for the storage account, then rehydrating a blob with Set Blob Tier can result in a scenario where the lifecycle policy moves the blob back to the archive tier after rehydration because the last modified time is beyond the threshold set for the policy.**

## Check your knowledge
Which access tier is considered to be offline and cannot be read or modified?

Archive

Which of the following storage account types supports lifecycle policies?

General Purpose v2