## Automate Deployments from VSTS to Octopus Deploy

[Octopus Deploy](https://Octopus.com) is an automated deployment server that makes it easy to automate deployment of ASP.NET web applications, Java applications, NodeJS application and custom scripts to multiple environments.

Visual Studio Team Services already includes a first-class, powerful release management capability that simplifies deployment of any application to any platform. But teams who prefer or already have chosen Octopus deploy, can use the **[Octopus Deploy Integration](https://marketplace.visualstudio.com/items?itemName=octopusdeploy.octopus-deploy-build-release-tasks)** extension available on the Visual Studio Marketplace that provides Build and Release tasks to integrate Octopus Deploy with VSTS and TFS.

This lab shows how you can integrate VSTS/TFS Team Build and Octopus to automate build and deployment application using a sample PHP application that will be deployed to an Azure App Service.

## Pre-requisites

1. **Microsoft Azure Account:** You will need a valid and active azure account for the lab.

2. You will need a **Visual Studio Team Services Account** and <a href="https://docs.microsoft.com/en-us/vsts/accounts/use-personal-access-tokens-to-authenticate">Personal Access Token</a>

## Setting up the Environment

1. Click on **Deploy to Azure** to provision Octopus Server.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVSTS-DevOps-Labs%2Foctopus%2Foctopus%2Farm%2520template%2Ftemplate.json"><img src="http://azuredeploy.net/deploybutton.png"></a> 

2. Provide **Resource group** and **Octopus DNS Name**, check the **Terms and Conditions** checkbox and click **Purchase**.

    <img src="images/DeployOctopus.png">

3. It takes approximately 15 minutes for the deployment to complete. Once the deployment is successful, resources including a Windows Server 2012 VM which hosts the **Octopus Deploy server**, an **Azure App Service** for the application deployment, a **SQL Server** with a **SQL Database** PaaS that acts as the backend of Octopus server is configured.

    <br/>

    <img src="images/Resources.png">

4. Click the **octopus-vm** in the **Overview** section and note down the **Subscription ID**, **DNS name**. We will need these values in later exercises of this lab.


   <img src="images/A3.png">

## Setting up the VSTS project

1. Use <a href="https://vstsdemogenerator.azurewebsites.net/?TemplateId=77370&name=octopus" target="_blank">VSTS Demo Data Generator</a> to provision the project on your VSTS account. 

   ![](images/1.png)

2. Provide the Project Name, the Octopus URL (VM's DNS URL) that was created previously, [API Key](https://octopus.com/docs/how-to/how-to-create-an-api-key) and click on Create Project. Once the project is provisioned, click the URL to navigate to the project.

   ![](images/DemoGen.png) 

>Note: This URL will automatically select Octopus template in the demo generator. If you want to try other projects, use this URL instead - https://vstsdemogenerator.azurewebsites.net/

## Exercise 1: Configure Deployment Target in Octopus Server

Let us create a deployment **environment** in Octopus server and link to Azure using **Management Certificate**. **Environments** are  deployment targets consisting of machines or services used by Octopus Deploy to deploy software. With Octopus Deploy, you can deploy software to Windows servers, Linux servers, Microsoft Azure, or even an offline package drop.

Grouping your deployment targets by environment lets you define your deployment processes and have Octopus deploy the right versions of your software to the right environments at the right time.

In this lab, we are using Azure App Service as the deployment target.  

1. Open a browser, enter the Octopus server URL using the VM's DNS name and below credentials:

   - **Octopus Server URL** : http://<your_vm_dns_name>.\<location>.cloudapp.azure.com
   - **Username**: admin
   - **Password**: P2ssw0rd@123

   <br/>

   <img src="images/O1.png">

2. Click **Create environment** to go into the *Infrastructure* page. Once inside, click **Add Environment**. 

   <img src="images/Create Environment.png">

   <br/>

   <img src="images/Add Environment.png">

3. Provide the environment name and click **Save**.

   <img src="images/DevEnvironment.png">

4. Octopus Deploy provides first-class support for deploying Azure Cloud Services and Azure Web Applications. To deploy software to Azure, you must add your Azure subscription to Octopus Deploy, and then use the built-in step templates to deploy to the cloud. Once the environment is created, click on **Accounts**. 

   <img src="images/Dev.png">


5. Click on **ADD ACCOUNT** to link your Azure subscription to the created environment.

   
   <img src="images/Add Account.png">

6. Octopus Deploy authenticates with Azure in one of two methods: using a **Management Certificate** or a **Service Principal**. To deploy to Azure Resource Manager (ARM), Octopus requires [**Azure Service Principal Account**](https://octopus.com/docs/infrastructure/azure/creating-an-azure-account/creating-an-azure-service-principal-account). 

   [**Azure Management Certificate**](https://octopus.com/docs/infrastructure/azure/creating-an-azure-account/creating-an-azure-management-certificate-account) is used by Octopus to deploy to Cloud Services and Azure Web Apps.

    Enter the following details -

    - **Name**: Provide the account name (any name)
    - **Subscription ID**: Your [Azure Subscription ID](https://blogs.msdn.microsoft.com/mschray/2016/03/18/getting-your-azure-subscription-guid-new-portal/)
    - **Authentication Method**: Choose **Use Management Certificate**

   <br/>

   <img src="images/Create Account.png">

7. Click **Save** and notice that a management certificate is generated. Download this certificate.

   <img src="images/Download Certificate.png">

8. To upload the certificate in Azure, go to [Azure Portal](https://portal.azure.com) and search for **Subscriptions**.

   <img src="images/O8.png">

9. Click on your Subscription.

   <img src="images/O9.png">

10. Scroll down and click **Management certificates**.

    <img src="images/O10.png">

11. Click **Upload** to upload the certificate which you downloaded in the **step 7**.

    <img src="images/O11.png">

    <br/>

    <img src="images/O12.png"  height="400px">

12. Once the certificate is uploaded successfully, go back to Octopus portal and click **Save and Test**. If the test succeeds, you should be able to configure Octopus to deploy anything to Azure.

    <img src="images/Verification Success.png">

## Exercise 2: Create Project in Octopus

Let us create a project in Octopus to deploy the package to **Azure App Service**. A [**Project**](https://octopus.com/docs/deployment-process/projects) is a collection of deployment steps and configuration variables that define how your software is deployed.

1. Go to Octopus dashboard and click **Create a project**.

   <img src="images/Project.png">

2. Click on **ADD PROJECT**, provide the project name, description and click on **SAVE**.

   <img src="images/Add Project.png">

   <br/>

   <img src="images/PUProject.png">

3. Once the project is created, click **Define your deployment process**. The [deployment process](https://octopus.com/docs/deploying-applications/deployment-process) is like a recipe for deploying your software.

   <img src="images/Define Process.png">

4. Click on **ADD STEP** to see a list of built-in step templates, custom step templates, and community contributed step templates.

   <img src="images/Add Step.png">

5. **Search** for **Azure Web App** template and click **Add**.

   <img src="images/AddWebAppStep.png">

6. Populate the step template with required details -

   - **Step Name**                   : A short, unique name for the template.
   - **Package ID**                  : PHP (if you are providing different package ID, update it in **Package PHP** task of the build definition)
   - **Azure account** & **Web App** : Select from the dropdown


   <img src="images/PkgID.png">

   <br/>

   <img src="images/Azure.png">

7. Clicking **Save** should define the project creation and its deployment process.

## Exercise 3: Link VSTS and Octopus Deploy Server

In this exercise, we will create an **API** key in Octopus. This key is required to link VSTS with Octopus. API keys allow you to access Octopus Deploy from VSTS without having to use your username and password. 

1. On the top right corner of Octopus portal, click on the currently logged in user and click **Profile**.

   <img src="images/userprofile.png">

2. In My Profile page click on **My API Keys** and click on **New API Key** to create a new key.



   <img src="images/API Key.png">

2. Give the **purpose** and click **Generate New**.

   <img src="images/Generate new.png">

3. Note down the API Key.

   <img src="images/Key.png">


> In this lab, **VSTS Demo Generator** provides an automated way of linking Octopus Deploy with VSTS. Instead, if you want to link manually, then you can follow the below steps to achieve the same end result.

4. Go to **VSTS project**, click on gear <img src="images/gear.png"> icon --> **Services --> + New Service Endpint**, scroll down and select **Octopus Deploy**

   <img src="images/Endpoint.png">


6. Provide **Connection name**, **Octopus server URL** and **API Key** and click OK. 

   <img src="images/endpointName.png">

7. We will see the service endpoint created.

   <img src="images/EndpointSuccess.png">

## Exercise 4: Triggering CI-CD

In this exercise, we will package PHP application and push the package to Octopus Server.

1. Go to **Builds** under **Build and Release** tab and click on **Octopus** build definition.

   <img src="images/Build Definition.png"> 
 
2. **Edit** the build definition to update Octopus server endpoint.

   <img src="images/EditBD.png">

3. In **Push Packages to Octopus** task, update **Octopus Deploy Server** field with the created endpoint value.

   > **Note** : You will encounter an error - **TFS.WebApi.Exception: Page not found** for Azure tasks in the release definition. This is due to a recent change in the VSTS Release Management API. While we are working on updating VSTS Demo Generator to resolve this issue, you can fix this by typing a random text in the **Azure Subscription** field and click the **Refresh** icon next to it. Once the field is refreshed, you can select the endpoint from the drop down.

   <img src="images/QBuild.png">

2. In **Create Octopus Release** task, update **Octopus Deploy Server** field with the created endpoint value and **Project** fields.

    <img src="images/Update1.png">

3. In **Deploy Octopus Release** task, update **Octopus Deploy Server** field with the created endpoint value, choose the appropriate values from the drop down for fields - **Project**  and **Deploy to Environments**.

    <img src="images/Update.png"> 

3. Save the build definition.

    <img src="images/Save.png">

  <table width="100%">
     <thead>
      <tr>
         <th width="50%"><b>Tasks</b></th>
         <th><b>Usage</b></th>
      </tr>
     </thead>
     <tr>
        <td><img src="images/octopuspackage.png"><b>Package Application</b></td>
        <td> We will package the PHP source code into a zip file with the version number</td>
     </tr>
     <tr>
      <td><img src="images/copyfiles.png"><b>Copy Files</b></td>
      <td> The Copy Files task will copy the generated package to artifacts directory in VSTS</td>
     </tr>
      <tr>
        <td><img src="images/pushpackage.png"><b>Push packages to Octopus</b></td>
        <td>The copied package will be pushed to Octopus server from VSTS artifacts directory</td>
     <tr>
      <td><img src="images/createoctopus.png"><b>Create Octopus Release</b></td>
      <td>Automates the creation of release in Octopus server</td>
     </tr>
     <tr>
      <td><img src="images/releaseoctopus.png"><b>Deploy Octopus Release</b></td>
      <td>Automates the deployment of release in Octopus server</td>
     </tr>

  </table>
<br/>






4. Go to **Code** tab and edit the file **functions.php**

    <img src="images/EditFile.png">

5. Update the **line 41** as shown, change the title to - **PHP DevOps Using VSTS, Octopus and Azure** and **commit** the changes.

    <img src="images/EditCommit.png">

6. Go to **Build** tab, you will see in-progress build.

    <img src="images/BuildProgress.png">

7. Once the build completes, go to Octopus portal project dashboard. You will see the release completion in Octopus.

    <img src="images/CD-Octopus.png">

8. Go to Azure Web App from your **[Azure Portal](https://portal.azure.com)** and click on **Browse**.

   <img src="images/Browse.png">

9. You will see the PHP application up and running.

   <img src="images/Changes.png">
   
## Summary

We can integrate Octopus with VSTS for delpoying applications to Azure.

## Feedback

Please email [us](mailto:devopsdemos@microsoft.com) if you have any feedback on this lab.