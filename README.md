
# AKS Periscope
Quick troubleshooting for your Azure Kubernetes Service (AKS) cluster.

# Overview
Hopefully most of the time, your AKS cluster is running happily and healthy. However, when things do go wrong, AKS customers need a tool to help them diagnose and collect the logs necessary to troubleshoot the issue. It can be difficult to collect the appropriate node and pod logs to figure what's wrong, how to fix the problem, or even to pass on those logs to others to help. 

AKS Periscope allows AKS customers to run initial diagnostics and collect and export the logs (like into an Azure Blob storage account) to help them analyze and identify potential problems or easily share the information to support to help with the troubleshooting process with a simple az aks kollect command. These cluster issues are many times are caused by wrong configuration of their environment, such as networking or permission issues. This tool will allow AKS customers to run initial diagnostics and collect logs and custom analyses that helps them identify the underlying problems.

![Architecture](https://user-images.githubusercontent.com/33297523/64900285-f5b65c00-d644-11e9-9a52-c4345d1b1861.png)

Raw Logs and metrics from AKS cluster are collected and basic diagnostic signals are generated by analyzing these raw data. More advanced diagnostic signals can be further generated by analyzing multiple basic diagnostic signals.

![Signals](https://user-images.githubusercontent.com/33297523/68249891-90dc0a00-ffd4-11e9-9eeb-fe9f35cbd173.png)

# Data Privacy and Collection
AKS Periscope runs on customer's agent pool nodes, and collect VM and container level data. It is important that the customer is aware and gives consent before the tool is deployed/information shared. Microsoft guidelines can be found in the link below:

https://azure.microsoft.com/en-us/support/legal/support-diagnostic-information-collection/


# Compatibility
AKS Periscope currently only work on Linux based agent nodes. Please see https://github.com/PatrickLang/logslurp for Windows based agent nodes. 


# Current Feature Set
It collects the following logs and metrics:

1. Container logs (by default all containers in the `kube-system` namespace, can be config to take other namespace/containers).
2. Docker and Kubelet system service logs.
3. Network outbound connectivity, include checks for internet, API server, Tunnel, Azure Container Registry and Microsoft Container Registry.
4. Node IP tables.
5. All node level logs (by default cluster provision log and cloud init log, can be config to take other logs).
6. VM and Kubernetes cluster level DNS settings.
7. Describe Kubernetes objects (by default all pods/services/deployments in the `kube-system` namespace, can be config to take other namespace/objects).
8. Kubelet command arguments.
9. System performance (kubectl top nodes and kubectl top pods).

It also generates the following diagnostic signals:

1. Network outbound connectivity, reports the down period for a specific connection.
2. Network configuration, includes Network Plugin, DNS, and Max Pods per Node settings.


# User Guide

AKS Periscope can be deployed by using Azure Command-Line tool (CLI). The steps are:

0. If CLI extension aks-preview has been installed previously, uninstall it first.
```
az extension remove --name aks-preview
``` 

1. Install CLI extension aks-preview.
```
az extension add --name aks-preview
``` 

2. Run `az aks kollect` command to collect metrics and diagnostic information, and upload to an Azure storage account. Use `az aks kollect -h` to check command details. Some useful examples are also listed below:

    1. Using storage account name and a shared access signature token with write permission
    ```
    az aks kollect
    -g MyResourceGroup
    -n MyManagedCluster
    --storage-account MyStorageAccount
    --sas-token "MySasToken"
    ```

    2. Using the resource id of a storage account resource you own.
    ```
    az aks kollect
    -g MyResourceGroup
    -n MyManagedCluster
    --storage-account "MyStorageAccountResourceId"
    ```

    3. Using a pre-setup storage account ((https://docs.microsoft.com/en-us/azure/azure-monitor/platform/diagnostic-logs-stream-log-store) in diagnostics settings for your managed cluster.
    ```
    az aks kollect
    -g MyResourceGroup
    -n MyManagedCluster
    ```

    4. Customize the container logs to collect. Its value can be either all containers in a namespace, for example, kube-system, or a specific container in a namespace, for example, kube-system/tunnelfront.
    ```
    az aks kollect
    -g MyResourceGroup
    -n MyManagedCluster
    --container-logs "mynamespace1/mypod1 myns2"
    ```

    5. Customize the kubernetes objects to collect. Its value can be either all objects of a type in a namespace, for example, kube-system/pod, or a specific object of a type in a namespace, for example, kube-system/deployment/tunnelfront.
    ```
    az aks kollect
    -g MyResourceGroup
    -n MyManagedCluster
    --kube-objects "mynamespace1/service myns2/deployment/deployment1"
    ```

    6. Customize the node log files to collect.
    ```
    az aks kollect
    -g MyResourceGroup
    -n MyManagedCluster
    --node-logs "/var/log/azure-vnet.log /var/log/azure-vnet-ipam.log"
    ```

All collected logs, metrics and node level diagnostic information is stored on host nodes under directory:
```
/var/log/aks-periscope
```
This directory is also mounted to container as:
```
/aks-periscope
```
After export, they will also be stored in Azure Blob Service under a container with its name equals to cluster API server FQDN. A zip file is also created for easy download.

Alternatively, AKS Periscope can be deployed directly with `kubectl`. See instructions in [Appendix].


# Programming Guide
Link to [Programming Guide].


# Feedback 
Feel free to contact aksperiscope@microsoft.com or open an issue with any feedback or questions about AKS Periscope. This is currently a work in progress, but look out for more capabilities to come!


# Contributing
This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.


[Programming Guide]: docs/programmingguide.md
[Appendix]: docs/appendix.md