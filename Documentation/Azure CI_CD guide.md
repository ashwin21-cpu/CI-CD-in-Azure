# CI/CD for Windows Containers with Azure DevOps
## Abstract
Azure DevOps enables you to host, build, plan and test your code with complimentary workflows. Using Azure Pipelines as one of these workflows allows you to deploy your application with CI/CD that works with any platform and cloud.
This sample application is a simple task-tracking app built with .NET Framework. The project has a workflow file, azure-pipelines.yaml, that is set up for Continuous Integration and Continuous Deployment.

## Prerequisites:
### 1. Create Resources
Create the following resources. You will need information from each resource that will be used in the pipeline file and stored as a variable. Create your Azure Container Registry first since you will need information from there before you can create your Azure Kubernetes Service.
*   Azure Container Registry (ACR). You can refer this [link](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal).
*   Azure Kubernetes Service (AKS cluster with windows node pool for deployments of application). You can create AKS cluster from this [link](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal).
*   Create a project inside the Azure DevOps organization in Azure [Azure DevOps Portal](https://dev.azure.com). Also import the github repository of your application within the project.

### 2. Add a Service Connection
Before you create your pipeline, you should first create your Service Connection since you will be asked to choose and verify your connection when creating your template. A Service Connection will allow you to connect to your ACR when using the task templates. You can create a new Service Connection following the directions [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#create-new). 

* #####  `Note: While creating service connection select Azure Resource Manager option.`

### 3. The Dockerfile
The samples below explain the associated Dockerfiles for the .NET Core sample applications linked above. If creating your own application, use the appropriate Dockerfile below and replace the directory paths to match your application.
For `.NET Core`, the nano server base image must be “1809” to be compatible with what Azure currently supports. Keep in mind this may change in the future.

![image](https://user-images.githubusercontent.com/86832286/155716984-72fef396-38d9-4cce-98ac-801b99aec6cc.png)


## Create the Pipeline
Once you have your repository created in Azure DevOps, or imported from GitHub, you can create your pipeline. On the left menu bar go to Pipelines and click the Create Pipeline button. The next screen will ask you where the code is to create the pipeline from. We already have our code imported, so we can choose Azure Repos Git to select your current repository. Since we are using Azure Kubernetes Service we can choose the Azure Kubernetes Service template that allows us to build and push an image to Azure Container Registry and deploy the application image in AKS cluster.

![image](https://user-images.githubusercontent.com/86832286/155716308-1f5f21f7-2db5-410d-8672-92d90377d36f.png)

Choose your `subscription` that you will be pushing your resources to, then pick your Container registry on the following screen. You will notice your Image Name and Dockerfile are pre-populated with a suggested name and path to your Dockerfile. You can leave those as it is, and click on the Validate and configure button to generate your azure-pipeline.yaml file.

## Secure Secrets with Variables

Variables are only accessible after your create the pipeline. Since we are using sensitive information that you don’t want others to access, we will use variables to protect our information. Create a variable by following the directions [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch). Add the variables below with your own secrets appropriate from each resource.

* `imageRepository`: 'image-name'
* `containerRegistry`: 'your-registry-name.azurecr.io'
* `applicationName`: 'app-name'

## Build the Pipeline
Once you have the necessary variables, you can start to add the tasks you need to complete the pipeline. Below is an explanation of the Docker tasks that were added to your pipeline from the AKS template with the addition of using ACR.

### Build and push your image to a registry
After your pipeline is generated from choosing the AKS configured template, you’ll notice a few things in the YAML build. Your image type is included and the build stages & task follow.

![image](https://user-images.githubusercontent.com/86832286/155988561-3ddecaa9-9055-4224-8468-9e790aafb355.png)

Note: Double check that your vmImageName = ‘windows-latest’ as it might default to ‘ubuntu-latest’.

### Azure Container Registry

![image](https://user-images.githubusercontent.com/86832286/155988713-dc9419a9-6927-4ba2-aca9-605c30b7e61a.png)

### Setup the Deploy Stage

This is done by using the first part of our yaml file and repurposing it to include our deployment tasks.

![image](https://user-images.githubusercontent.com/86832286/156116102-dcd1d8ee-194c-4379-bf91-5820d74fc20d.png)

### Deploy to Azure Kubernetes Service

Now that you have your image pushed to your registry and your deploy stage setup. You can push the container to Azure Kubernetes Service. If you haven’t already created your Azure Kubernetes Service in the Azure portal, you’ll need to do so now before you can proceed.

![image](https://user-images.githubusercontent.com/86832286/156117133-98ff64eb-bf38-4fc6-ac86-db71bbb3436d.png)

`Note:` The above steps include microsoft hosted Agent Pool. For self hosted Agent Pool follow the below step.

### Setup for Self-Hosted Agent Pool

For self-hosted Agent Pool please proceed to this [link](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops) 

* Also edit the azure-pipeline.yaml file as shown here. ![image](https://user-images.githubusercontent.com/82659622/156331877-173859ee-9819-42dd-aa45-351d9da1e37c.png)
* Replace the name of the self hosted Agent Pool and comment the vmImage at each occurence in the file.



## Deploy to Azure SQL Database

To deploy your SQL database to Azure, we will use a dacpac file or SQL scripts, and a connection string. The connection string can be found in the overview page of your Azure SQL database and you can create your dacpac by extracting the data using something like SQL Server Object Explorer in Visual Studio.

If you need to import a SQL database that you’d like to host on Azure, you have the option to add in the Azure SQL Database deployment task which can be found in the assistant we used earlier. Enter the following values for the parameters

![image](https://user-images.githubusercontent.com/82659622/157427871-69fbae1e-248d-4bdc-8edb-0d974a62b768.png)

Once created with the above parameters, the output should show as below.

![image](https://user-images.githubusercontent.com/82659622/157428520-57ccbff9-5be8-4092-8aab-350048119735.png)

#### Connection String
Our sample app has a dummy connection string in the appsettings.json that will need to be changed for local testing. Make sure the same connection string must be added in the `azure-pipeline.yaml` file within the SqlAzureDacpacDeployment@1 task.

When adding the values for the SQL Database parameters, you’ll want to choose Connection String as your Authentication Type and add in your connection string. You can use Variables in DevOps to hide the connection string in safe keeping.

### Creating a dacpac file in your project
As mentioned before, you’ll need to use either a dacpac file or set of SQL scripts to deploy your database schema. If you are using Visual Studio, it’s easy to create and add the needed dacpac file to run the action.

1. Connect your SQL Azure Database to Visual Studio
2. Right-click the data base and choose Extract Data-tier application
3. On the following window, choose the location at the same level of your github workflow file and click create.
    
  ![image](https://user-images.githubusercontent.com/82659622/157430098-c8b8d861-f56a-42c1-8302-d39d7276e9aa.png)
  
Your dacpac file should have been created and added to your project. The action finds your file under the dacpac-package parameter seen above.


### Summary

From here you are setup to continuously build your Windows Container application through Azure DevOps. Below you’ll see the final result of the workflow yaml file.

#### Full workflow file

The previous sections showed how to assemble the workflow step-by-step. The full `azure-pipelines.yaml` is below.

```bigquery
# Deploy to Azure Kubernetes Service
# Build and push image to Azure Container Registry; Deploy to Azure Kubernetes Service
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

# The branch that triggers the pipeline to start building
trigger:
- master

# The source used by the pipeline
resources:
- repo: self

variables:
  # Container registry service connection established during pipeline creation
  dockerRegistryServiceConnection: '<your-service-connection-number>'
  imageRepository: '<image-repository-name>'
  containerRegistry: '<your-container-registry>'
  dockerfilePath: '**/Dockerfile'
  tag: '$(Build.BuildId)'
  imagePullSecret: '<image-pull-secret>'

  # Agent VM image name
  vmImageName: 'windows-latest'

  # Name of the new namespace being created to deploy the PR changes.
  k8sNamespaceForPR: 'review-app-$(System.PullRequest.PullRequestId)'

stages:
- stage: Build
  displayName: Build stage
  jobs:
  - job: Build
    displayName: Build
    pool: 
     vmImage: $(vmImageName)      
    steps:
    - task: Docker@2
      displayName: Build and push an image to container registry
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)

    - upload: manifests
      artifact: manifests

- stage: Deploy
  displayName: Deploy stage
  dependsOn: Build

  jobs:
  - deployment: Deploy
    condition: and(succeeded(), not(startsWith(variables['Build.SourceBranch'], 'refs/pull/')))
    displayName: Deploy
    pool: 
     vmImage: $(vmImageName)
    environment: 'Taskapp.default'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@0
            displayName: Create imagePullSecret
            inputs:
              action: createSecret
              secretName: $(imagePullSecret)
              dockerRegistryEndpoint: $(dockerRegistryServiceConnection)

          - task: KubernetesManifest@0
            displayName: Deploy to Kubernetes cluster
            inputs:
              action: deploy
              manifests: |
                $(Pipeline.Workspace)/manifests/deployment.yml
                $(Pipeline.Workspace)/manifests/service.yml
              imagePullSecrets: |
                $(imagePullSecret)
              containers: |
                $(containerRegistry)/$(imageRepository):$(tag)


  - deployment: DeployPullRequest
    displayName: Deploy Pull request
    condition: and(succeeded(), startsWith(variables['Build.SourceBranch'], 'refs/pull/'))
    pool: 
     vmImage: $(vmImageName)

    environment: 'Taskapp.$(k8sNamespaceForPR)'
    strategy:
      runOnce:
        deploy:
          steps:
          - reviewApp: default

          - task: Kubernetes@1
            displayName: 'Create a new namespace for the pull request'
            inputs:
              command: apply
              useConfigurationFile: true
              inline: '{ "kind": "Namespace", "apiVersion": "v1", "metadata": { "name": "$(k8sNamespaceForPR)" }}'

          - task: KubernetesManifest@0
            displayName: Create imagePullSecret
            inputs:
              action: createSecret
              secretName: $(imagePullSecret)
              namespace: $(k8sNamespaceForPR)
              dockerRegistryEndpoint: $(dockerRegistryServiceConnection)

          - task: KubernetesManifest@0
            displayName: Deploy to the new namespace in the Kubernetes cluster
            inputs:
              action: deploy
              namespace: $(k8sNamespaceForPR)
              manifests: |
                $(Pipeline.Workspace)/manifests/deployment.yml
                $(Pipeline.Workspace)/manifests/service.yml
              imagePullSecrets: |
                $(imagePullSecret)
              containers: |
                $(containerRegistry)/$(imageRepository):$(tag)

          - task: Kubernetes@1
            name: get
            displayName: 'Get services in the new namespace'
            continueOnError: true
            inputs:
              command: get
              namespace: $(k8sNamespaceForPR)
              arguments: svc
              outputFormat: jsonpath='http://{.items[0].status.loadBalancer.ingress[0].ip}:{.items[0].spec.ports[0].port}'
  
              
          # Getting the IP of the deployed service and writing it to a variable for posing comment
          - script: |
              url="$(get.KubectlOutput)"
              message="Your review app has been deployed"
              if [ ! -z "$url" -a "$url" != "http://:" ]
              then
                message="${message} and is available at $url.<br><br>[Learn More](https://aka.ms/testwithreviewapps) about how to test and provide feedback for the app."
              fi
              echo "##vso[task.setvariable variable=GITHUB_COMMENT]$message"
              
- stage: SqlDeployment
  displayName: Sql DB Deployment
  jobs:
  - job: SqlDeployment
    displayName: Sql DB Deployment
    pool:
      vmImage: $(vmImageName)
    steps:
    - task: SqlAzureDacpacDeployment@1
      inputs:
        azureSubscription: 'AzureDemoSC'
        AuthenticationType: 'connectionString'
        ConnectionString: '<azureSQLConnectionString>'
        deployType: 'DacpacTask'
        DeploymentAction: 'Publish'
        DacpacFile: '$(System.DefaultWorkingDirectory)/**/data.dacpac'

```


