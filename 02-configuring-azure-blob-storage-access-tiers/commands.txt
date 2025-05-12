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
