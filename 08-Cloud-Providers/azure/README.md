# Azure Cheatsheet

## Core Services Overview
- Azure VMs: Virtual machines
- Azure Blob Storage: Object storage
- Azure Active Directory (AAD): Identity
- Azure Kubernetes Service (AKS): Managed Kubernetes
- Azure SQL/Database services

## Azure CLI

### Install & Login
```bash
az login
az account set --subscription "SUBSCRIPTION_NAME"
```

### Resource Groups
```bash
az group create --name rg-app --location eastus
az group list -o table
```

## Virtual Machines
```bash
# Create VM
az vm create \
  --resource-group rg-app \
  --name vm1 \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys

# Open port 80
az vm open-port --port 80 --resource-group rg-app --name vm1
```

## Blob Storage
```bash
# Create storage account and container
az storage account create -n mystorage$RANDOM -g rg-app -l eastus --sku Standard_LRS
ACCOUNT=$(az storage account list -g rg-app --query "[0].name" -o tsv)
KEY=$(az storage account keys list -n $ACCOUNT -g rg-app --query "[0].value" -o tsv)
az storage container create -n mycontainer --account-name $ACCOUNT --account-key $KEY

# Upload
az storage blob upload --account-name $ACCOUNT --account-key $KEY -f file.txt -c mycontainer -n file.txt
```

## Azure Active Directory (AAD)
```bash
# List users and groups
az ad user list --query "[].{UPN:userPrincipalName}" -o table
az ad group list --query "[].displayName" -o table
```

## AKS
```bash
# Create AKS
az aks create -g rg-app -n aks1 --node-count 2 --generate-ssh-keys

# Get credentials
az aks get-credentials -g rg-app -n aks1

# Validate
kubectl get nodes -o wide
```

## Networking Basics
- VNets, Subnets, NSGs, UDRs, Public IPs, Load Balancers

## Best Practices
- Use Managed Identity instead of client secrets
- Resource tagging and RBAC
- Use Azure Monitor and Log Analytics
- Use IaC (Bicep/Terraform)
