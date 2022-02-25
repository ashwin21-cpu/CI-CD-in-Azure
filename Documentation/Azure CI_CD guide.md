# CI/CD for Windows Containers with Azure DevOps

Aure DevOps enables you to host, build, plan and test your code with complimentary workflows. Using Azure Pipelines as one of these workflows allows you to deploy your application with CI/CD that works with any platform and cloud.
This sample application is a simple task-tracking app built with .NET Framework. The project has a workflow file, azure-pipelines.yaml, that is set up for Continuous Integraion and Continuous Deployment.

## Prerequisites:
### 1. Create Resources
Create the following resources. You will need information from each resource that will be used in the pipeline file and stored as a variable. Create your Azure Container Registry first since you will need information from there before you can create your Azure Kubernetes Service.
*   Azure Container Registry (ACR). You can refer this [link](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-portal).
*   Azure Kubernetes Service (AKS cluster with windows node pool for deployments of application). You can create AKS cluster from this [link](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal).
*   Create a project inside the Azure DevOps organization in Azure [Azure DevOps Portal](https://dev.azure.com). Also import the github repository of your application within the project.

### 2. Add a Service Connection
Before you create your pipeline, you should first create your Service Connection since you will be asked to choose and verify your connection when creating your template. A Service Connection will allow you to connect to your ACR when using the task templates. You can create a new Service Connection following the directions [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#create-new).

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

* `imageRepository`: image-name
* `containerRegistry`: ‘your-registry-name.azurecr.io’
* `applicationName`: app-name

## Build the Pipeline
Once you have the necessary variables, you can start to add the tasks you need to complete the pipeline. Below is an explanation of the Docker tasks that were added to your pipeline from the AKS template with the addition of using ACR.

### Build and push your image to a registry
After your pipeline is generated from choosing the AKS configured template, you’ll notice a few things in the YAML build. Your image type is included and the build stages & task follow.

![image](https://user-images.githubusercontent.com/86832286/155719677-eb632f44-de30-4fad-8604-f14e7cdb0443.png)


