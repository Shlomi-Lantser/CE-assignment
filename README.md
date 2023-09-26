# CE-Assignment



## Table of Contents
- [Introduction](#introduction)
- [Resources](#Resources)
- [Getting Started](#getting-started)
  - [Provided Resources](#Provided-resources)
  - [Steps of implementation](#Steps-of-implementation)
- [Testing](#Testing)

## Introduction

 In this assignment, we'll go through the process of creating and managing an Azure Kubernetes Service (AKS) cluster, configuring virtual networks (VNets), deploying Nginx Ingress Controller for routing internal traffic, and launching an AKS-helloworld application within the AKS cluster. Additionally, we'll set up an Azure Application Gateway to act as a layer 7 load balancer for routing external traffic to the Ingress Controller.

## Resources

- **Azure AKS Cluster**: Create and manage a highly available AKS cluster.
- **Two VNets**: Configured two VNets for a more secure and isolated network architecture.
- **Nginx Ingress Controller**: Used Helm charts to install the Nginx Ingress controller for routing internal traffic.
- **AKS-helloworld Application**: A sample application deployed in the AKS cluster using helm charts.
- **Azure Application Gateway** : An layer 7 load balancer associated to PIP for routing external traffic to the ingress controller

## Getting Started

### Provided resources

- An Azure subscription.
- A resource group named *FILL*

### Steps of implementation

1. Created the aks-vnet in the given resource group and its VNet:

   ```bash
   az network vnet create -g aks-lab \  
   --name aks-vnet --address-prefix 10.224.0.0/12 \
   --subnet-name aks-subnet \
   --subnet-prefix 10.224.0.0/16
   
2. Pulled the subnet id by query :
    ```bash
    subnetId=$(az network vnet show -g aks-lab -n aks-vnet --query "subnets[?name=='aks-subnet'].id" --output tsv)
    
3. Deployed the AKS cluster within the VNet i created in the (1) step:
    ```bash
    az aks create -g aks-lab \
    -n aks-lab-aks-cluster-westeu \
    --enable-managed-identity \
    --node-count 1 \
    --node-resource-group MC_aks-lab_aks-cluster-westeu \
    --generate-ssh-keys --vnet-subnet-id $subnetId
    
4. Added the nginx-ingress helm repository as explained in the document:
    ```bash
    helm repo add nginx-stable https://helm.nginx.com/stable
    ```
    ```bash
    helm repo update
    
5. Edited the values file of the ingress helm chart:  
There are couple of ways to change helm charts values and costumize them i chose for ingress the custom values file option
    ```bash
    controller:
        service:
            loadBalancerIP: 10.224.0.42
            annotations:
                service.beta.kubernetes.io/azure-load-balancer-internal: "true"
    ```
    Explanation :
    * loadBalancerIP - I chose 10.224.0.42 as it inside my CIDR range of the aks-vnet so the ip of the ingress controller service will be internal VNet ip
    * annotations : chose this option to restrict access to the services to internal users with no external access.
    
6. Added the aks-hellowold helm chart:
    ```bash
    helm repo add azure-samples https://azure-samples.github.io/helm-charts/
    ```

7. Edited the aks-helloworld helm chart values with the custom values file option:
    ```bash
    helm show values azure-samples/aks-helloworld > aks-helloworld-values.yaml
    nano values.yaml
    ```
    changed the title value and installed the helm chart using the values file i created
    ```bash
    helm install aks-helloworld azure-samples/aks-helloworld -f aks-helloworld-values.yaml
    ```
    
8. Created a rule of the nginx ingress controller using yaml file to configuration:  
    [ingress-rule.yaml](https://github.com/Shlomi-Lantser/CE-assignment/blob/main/yaml-files/ingress-rule.yaml)
    
9. Used the kubectl apply command to apply the rule:
    ```bash
    kubectl apply -f ingress-rule.yaml
    ```
10. Created a resource group named : app-gw-rg
    ```bash
    az group create --name aks-app-gw-rg --location westeurope
    ```
    
12. Deployed the hub vnet within the app-gw-rg resource group:
    ```bash
    az network vnet create -g aks-app-gw-rg \
    --name Hub-vnet \
    --address-prefix 10.4.0.0/16 \
    --subnet-name app-gw-subnet \
    --subnet-prefix 10.4.0.0/24 \
    --location westeurope

13. Create a peering for both VNets to allow communication:
    ```bash
    az network vnet peering create -g aks-app-gw-rg \
    --vnet-name Hub-vnet \
    --name hub2aks \
    --remote-vnet $(az network vnet show -g aks-lab -n aks-vnet --query id -o tsv) \
    --allow-vnet-access
    ```
    ```bash
    az network vnet peering create -g aks-lab \
    --vnet-name aks-vnet \
    --name aks2hub \
    --remote-vnet $(az network vnet show -g aks-app-gw-rg -n Hub-vnet --query id -o tsv) \
    --allow-vnet-access
    ```
14. Added the app-gw-subnet to the AKS route table:
    ```bash
    routeTableId=$(az network route-table show -g MC_aks-lab_aks-cluster-westeu --name aks-agentpool-34800524-routetable --query id -o tsv)
    ```
    ```bash
    az network vnet subnet update -g aks-app-gw-rg \
    --vnet-name Hub-vnet \
    --name app-gw-subnet --route-table $routeTableId
    ```
      
15. Deployed the Application Gateway within the Hub-Vnet by using the following steps:
    * dsadas
    * dsadsa
    * fsfds

15.Created an costum health probe with the following settings :



### Testing

Created a record in my computer hosts file that resolves my chosen host : myapp.com  
```bash
app-gw-pip myapp.com  
```
And this is the result:

![image](https://github.com/Shlomi-Lantser/CE-assignment/assets/92504985/2b679db8-bcc2-4a60-b7d3-c644c5a435a1)

