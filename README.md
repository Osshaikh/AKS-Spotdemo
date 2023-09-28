# AKS-Spotdemo
Using Spot VM's as nodepool in AKS with High Avaliblity Workaround

Azure Spot VM's is great way utilize azure spare compute capacity up to 90% off for your noncritical workloads.

Spot VM's are primarily used for workloads that can tolerate interruptions, such as batch processing and scientific simulation, big data workloads, such as Hadoop clusters or Spark jobs or even for burstable workload for intermittent traffic spikes.

Also, you could query azure spot VM eviction or spot pricing for a specific VM series in a region easily using Kusto queries via Az Resource Graph explorer, based on that you could select VM SKU which has least eviction rate probability & price.

Sample Queries:  https://learn.microsoft.com/en-us/azure/virtual-machines/spot-vms#azure-resource-graph


Inside Instance

To Prepare individual VM instances for interruption, you could query IMDS (instance Metadata service) REST API which is accessible from within VM that also use Az Scheduled events to notify about upcoming events on VM instance based on event type i.e., freeze, reboot, redeploy, preempt(spot) and for Spot VM platform gives 30 seconds notification prior to eviction.

following is example to avoid interruption on workloads when spot eviction notification is received, you could configure script in cron job to query IMDS API every minute.


#!/bin/bash
# Set metadata URL
metadataURL=" http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01"

# Get scheduled events
response=$(curl -H Metadata:true --noproxy "*" $metadataURL)

# Check for reboot, freeze or terminate eventTypes
if [[ $response == *"\"eventType\":\"preempt\""* ]] || [[ $response == *"\"eventType\":\"reboot\""* ]]; then

# Insert your code here to handle interruptions ,

else
 # Do nothing if event Type is blank or not one of the above

 echo "No action needed."
fi

Outside Instance

Activity logs capture spot eviction notifications with operator name as "EvictSpotVM", with that information an alert can be configured and can be used trigger point to integrate with Az function or Automation runbook to minimize the impact of interruption.




AKS VMSS also supports Spot nodepools :

Using 'Cluster Auto scaler Expander Priority' Addon priority can be configured for spot node pool bases on that spot node pool would be preferred for all your deployment unless not available.


Reference link for CA expander feature:  https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-expanders

Configure Cluster Autoscaler ExpanderFeaure:  https://learn.microsoft.com/en-us/azure/aks/cluster-autoscaler
![image](https://github.com/Osshaikh/AKS-Spotdemo/assets/44756471/28891fc8-ad9e-449b-a586-47dcf7618a68)

