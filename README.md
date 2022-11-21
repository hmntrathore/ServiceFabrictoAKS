# Migration journey from Azure Service Fabric container to AKS
Writing this document to capture the journey to migrate from ASF - containers to AKS continers for a 8 years old legacy app

## Discover your application: 
Use tools like Cloud Pilot to discover ASP.NET application and the other applications components.

Video link of the tool-> https://www.youtube.com/watch?v=cO27cq9IiuE&list=PL8ZFxfkhI2ewuyjmmcls7_xTJX1ciEARl
Check the compatibility from the report and fix the code if anything is mandatory.

If the code is legacy and you don’t have enough time to rewrite the code from .net framework to .net core, then first upgrade the code –

  a.	from .net framework current version to the latest version and deploy in AKS Windows containers. 
  b.	from .net core to .net 6.0 and deploy in AKS Linux containers.
  
For deciding  the .net framework version use the link-
  https://learn.microsoft.com/en-us/lifecycle/products/microsoft-net-framework
For upgrading to latest version use the below link-

 https://docs.microsoft.com/en-us/dotnet/standard/analyzers/portability-analyzer
 https://dotnet.microsoft.com/en-us/platform/upgrade-assistant/tutorial/install-upgrade-assistant

## Developer steps
Changes must be made in the new or existing repository as per the organization process.
## Migration from Framework 4.6.1 to 4.x
•	Change the target framework from “<TargetFrameworkVersion>v4.6.1</ TargetFrameworkVersion >“ to “<TargetFrameworkVersion >v4.x</ TargetFrameworkVersion >“ for every project in the solution that you are targeting.
<br>
•	If your repo has multiple solutions and they are configured to build the pipeline, then update them all together to use the new version. 
<br>
•	Some NuGet will give issues, reinstall them again, fix the issues locally. It will not affect the pipeline build because the NuGet is installed every time the pipeline runs.
<br>
•	If the AKS deployment folder is not available, create and add the yaml files to the AKS deployment folder. Change Docker image for latest version. 
<br>
•	Consider Stateless microservices. 
<br>
•	Data processing logic using Persistent Storage must be validated. Use Volumes, Persistent Volumes, Storage Classes, Persistent volume claims.
<br>
•	Stateful microservices are not in scope of migration.
<br>
•	AKS container hosted sites interact with ASF hosted WCF services.
<br>
•	Changes related to Common DLL will be required  to be published (nuget package /physical dll references)
<br>
•	The Service Fabric programming models (reliable services, reliable actors, Guest executables will need revisit /redesign.

## DevOps practices
•	All developers have installed the latest version of Visual Studio 2022 and appropriate .NET SDK 1. 
<br>
•	While building the solution locally, you may face issues related to NuGet not able to install after the upgrade, just restart the visual studio and load the solution again.
<br>
•	If project dlls are pushed to common folder, the consumer project will fail until the related common DLL folders are having correct versions. 
<br>
•	You may find “DLL not found issue” with the upgrade, to solve this please remove the related binding from the web.config and run the build again.

## Infra Changes 
•	Create New Subnet for AKS inside the existing Vnet. Subnet size-should be greater or equal to number of pods
<br>
•	Create New AKS Cluster 
<br>
•	Create node pool with ephemeral disk and availability zones. Test throughput
<br>
•	Avoid using custom dns
<br>
•	Use Static public IP address
<br>
•	Azure vNET integration via Azure CNI
<br>
•	Nginx ingress controller for routing is used
<br>
•	NSG, front door + WAF implemented in prevention mode, no azure firewall, traffic comes is filtered at nsg level.
<br>
## Code Changes 
•	Add K8s.deployment folder in Repo.
<br>
•	Change Environment variable to Config.yaml (refer to Service manifest used for ASF all used env variables from service manifest file must be moved to config.yaml)
<br>
•	If you are using 2 docker files to run simultaneously ASF & AKS, then do the below changes to upgrade image to support 4.x 
<br>
•	Add new Docker file for AKS
<br>
•	Change existing Docker file for ASF till it is deployed in AKS production.
<br>
•	Use latest Windows server 2022 Images 
<br>
## Pipeline Changes 
•	Add AKSConfigPrefix in Web.config
<br>
•	ADO pipeline changes related to FrameWork (Windows)  
<br>
•	Octopus Pipeline Changes Windows - Octopus Deploy Linux - Octopus Deploy.
<br>
•	All pipelines should be independently released so the apps team can take release any time. 
<br>
•	Disable DEV ADO and Octopus pipeline once they are good with aks
<br>
•	Carefully manage the dll chains. Use NuGet for package management.
<br>
•	Application team to communicate with all consumers about the New dll versions. Publish the documented process to consume the latest dlls for new changes. 
## AKS Changes
o	Resource Quotas/RBAC at namespace level
<br>
o	Review existing policy to handle AKS Configuration, maintain specific set of nodes type, number etc
<br>
o	Maintain segregation of Node pool- system & user, use combination of taints/tolerations, node affinity + pod affinity in specified namespace for user node pool
<br>
o	Use SLA backed AKS
<br>
o	Use secrets from keyvault with CSI driver, currently used through config.yaml
<br>
o	Optimize request and limit, after few deployments and testing are completed.
<br>
o	Communication from namespace A to namespace B can be restricted through network policies
<br>
o	Used ACR as image repository with service principal
<br>
## Migration Process for Framework Projects
The DevOps team can perform repository-wise migration. For framework projects update the framework version with 4.x on the existing dev repository. 
<br> Once the package is ready deploy it. Post deployment point the application on dev branch from ASF to AKS. Once the Developer provides sign-off on Dev, perform API health check, DevOps then updates the Release branch code with the latest framework and deploys the AKS and ASF on QC. In between if any features or bug fixing is required by the app team then they can work on the current repository and take release at any time. The old Deployment pipeline can be alive till production. Disable the Old Pipeline across all branch’s postproduction deployment.
<br>
 ** Steps are more or less similar for.NET Core 2.1/2.2/3.1 to .NET 6.0
<br>
*Reference link to follow  https://learn.microsoft.com/en-us/azure/migrate/tutorial-app-containerization-aspnet-kubernetes
## Challenges faced
•	Multiple build related errors during each project migration. Create KB article an document learning.
<br>
•	Multiple dependency- legacy dll consumption causing more dependency and migration is not straight forward without checking dependency. Use nuget package library.
<br>
•	Performance issues- with latest .Net framework version team expected a performance boost. Test and verify with tools.
<br>
•	Monitoring tools used like newrelic at infra and app level, Container insight is also used. At the node level and pod level monitoring data are checked.
<br>
