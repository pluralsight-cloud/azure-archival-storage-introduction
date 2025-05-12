# Commands and Code for Introduction to Azure Archival Storage

Keep in mind that you may need to change both the resource group name and the storage account names to suit your needs.

## Module 1

These commands assume you are creating a resource group of `az-cli-demo-rg` a Storage Account named `azcliaccesstierdemo` and a container within that storage account called `cli-demo-container`. 

Remember to replace the values with the values you are using in your Azure Account.

Create the Resource Group:

```
az group create \
  --name az-cli-demo-rg \
  --location westus2 \
  --only-show-errors
```

Create the Storage Account (This command can take ~20 seconds):

```
az storage account create \
  --name azcliaccesstierdemo \
  --resource-group az-cli-demo-rg \
  --location westus2 \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot \
  --only-show-errors
```

Create Container:

```
az storage container create \
  --name cli-demo-container \
  --account-name azcliaccesstierdemo \
  --only-show-errors
```

Create a new file:

```
echo "this is a hot file" > hotfile.txt
```

Upload a file to your storage account/container:

```
az storage blob upload \
  --container-name cli-demo-container \
  --name hotfile.txt \
  --file hotfile.txt \
  --account-name azcliaccesstierdemo \
  --only-show-errors
```

Show the blob in your new container:

```
az storage blob show \
  --container-name cli-demo-container \
  --name hotfile.txt \
  --account-name azcliaccesstierdemo \
  --only-show-errors
```

Show a subset of blob attributes with JQ:

```
az storage blob show \
  --container-name cli-demo-container \
  --name hotfile.txt \
  --account-name azcliaccesstierdemo \
  --output json \
  --only-show-errors | jq '{container, name, blobTier: .properties.blobTier, blobType: .properties.blobType}'
``` 

Create new files to test with:

```
echo "this is a cool file" > coolfile.txt
echo "this is a cold file" > coldfile.txt
echo "this is an archive" > archive.txt
```

Upload new files to the storage account/container in different access tiers:

```
az storage blob upload \
  --container-name cli-demo-container \
  --name coolfile.txt \
  --file coolfile.txt \
  --account-name azcliaccesstierdemo \
  --tier Cool \
  --only-show-errors

az storage blob upload \
  --container-name cli-demo-container \
  --name coldfile.txt \
  --file coldfile.txt \
  --account-name azcliaccesstierdemo \
  --tier Cold \
  --only-show-errors

az storage blob upload \
  --container-name cli-demo-container \
  --name archive.txt \
  --file archive.txt \
  --account-name azcliaccesstierdemo \
  --tier Archive \
  --only-show-errors
```

List out the contents of the Storage Account and container:

```
az storage blob list \
  --container-name cli-demo-container \
  --account-name azcliaccesstierdemo \
  --output table \
  --only-show-errors
```

Remove the files on the local system: 

```
rm hotfile.txt
rm coolfile.txt
rm coldfile.txt
rm archive.txt
```

Download files from the Storage Account:

```
az storage blob download \
  --container-name cli-demo-container \
  --name hotfile.txt \
  --file hotfile.txt \
  --account-name azcliaccesstierdemo \
  -o none \
  --only-show-errors

az storage blob download \
  --container-name cli-demo-container \
  --name coolfile.txt \
  --file coolfile.txt \
  --account-name azcliaccesstierdemo \
  -o none \
  --only-show-errors

az storage blob download \
  --container-name cli-demo-container \
  --name coldfile.txt \
  --file coldfile.txt \
  --account-name azcliaccesstierdemo \
  -o none \
  --only-show-errors
```

Attempt to download the archive file (this will fail):

```
az storage blob download \
  --container-name cli-demo-container \
  --name archive.txt \
  --file archive.txt \
  --account-name azcliaccesstierdemo \
  -o none \
  --only-show-errors
```

Confirm it didn't download with:

```
ls
```

## Module 2

Lifecycle Policy:

```
{
  "rules": [
    {
      "enabled": true,
      "name": "move-hot-to-cool",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "hotcontainer"
          ]
        }
      }
    }
  ]
}
```


These commands assume you have already created an account named `psdemoacct1` and container within that called `mixedcontainer`. 

You can refer to the first module commands on how to create Azure Storage Accounts and containers.

Remember to replace the values with the values you are using in your Azure Account.

Show all blobs in mixedcontainer:

```
az storage blob list --account-name psdemoacct1 --container-name mixedcontainer --output table
```

Describe Lifecycle Management Policies:

```
az storage account management-policy show --account-name psdemoacct1 --resource-group azure-storage-tiers
```

Output the current policy to JSON file:
```
az storage account management-policy show --account-name psdemoacct1 --resource-group azure-storage-tiers | jq .policy > temp-policy.json
```

Edit the file:

```
nano temp-policy.json
```

Set the new policy:

```
az storage account management-policy create \
  --account-name psdemoacct1 \
  --resource-group azure-storage-tiers \
  --policy @temp-policy.json
```

Check that it worked:

```
az storage account management-policy show --account-name psdemoacct1 --resource-group azure-storage-tiers
```