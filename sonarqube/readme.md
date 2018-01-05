## Managing Technical Debt using VSTS and SonarQube 

## Overview

Technical debt is the set of problems in a development effort that make forward progress on customer value inefficient. Technical debt saps productivity by making code hard to understand, fragile, time-consuming to change, difficult to validate, and creates unplanned work that blocks progress. Unless they are managed, technical debt can accumulate and hurt the overall quality of the software and the productivity of the development team in the long term

[SonarQube](https://www.sonarqube.org/) an open source platform for continuous inspection of code quality to perform automatic reviews with static analysis of code to:

- Detect Bugs
- Code Smells
- Security Vulnerabilities
- Centralize Quality

In this lab, you will learn how to setup SonarQube on Azure and to integrate Visual Studio Team Services

- Provision SonarQube server from an Azure template
- Setup SonarQube project
- Setup a VSTS project and CI build to integrate with SonarQube
- Analyze SonarQube reports

## Pre-requisites

1. **Microsoft Azure Account:** You need a valid and active azure account for the labs

2. You need a **Visual Studio Team Services Account** and <a href="https://docs.microsoft.com/en-us/vsts/accounts/use-personal-access-tokens-to-authenticate">Personal Access Token</a>

## Setting up the Environment

1. Click the **Deploy To Azure** button below to provision SonarQube Server on Azure VM.

   <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVSTS-DevOps-Labs%2Fmaster%2Fsonarqube%2Ftemplates%2Fazuredeploy.json"><img src="http://azuredeploy.net/deploybutton.png"></a>

    <img src="images/CustomDeployAzure1.png"></a> 

    <img src="images/CustomDeployAzure2.png"></a> 

  
   Provide the following parameters as shown.
   <table width="100%">
   <thead>
      <tr>
         <th width="50%"><b>Parameter Name</b></th>
         <th><b>Description</b></th>
         

   </thead>
   <tr>
      <td>Subscription Details</td>
      <td>Choose the active Azure subscription, create a new resource group along with the location of creation.</td>
      
   </tr>
 

   <tr>
      <td>SQ_VM_Name</td>
      <td>name of the VM where SonarQube will be installed</td>
      
   </tr>
   <tr>
      <td>SQ_DNS_NAME</td>
      <td><b>unique</b> dns name to be provided with the following pattern:- <b>^[a-z][a-z0-9-]{1,61}[a-z0-9]$</b> or it will throw an error. For ex: sonarqubedns</td>
      
   </tr>
   <tr>
      <td>SQ_VM_UserName</td>
      <td>local admin account for the SonarQube VM</td>
      
   </tr>
   <tr>
      <td>SQ_VM_UserPassword</td>
      <td>password for the SonarQube VM</td>
      
   </tr>
   <tr>
      <td>SQ_DBAdmin_UserName</td>
      <td>admin account for Azure SQL Server</td>
      
   </tr>
   <tr>
      <td>SQ_DBAdmin_Password</td>
      <td>password for Azure SQL Server</td>
      
   </tr>
   <tr>
      <td>_artifacts Location</td>
      <td>This value will be automatically generated. Leave the field as it is.</td>
      
   </tr>
   <tr>
      <td>_artifacts Location Sas Token</td>
      <td>The Sas Token will be automatically generated. Leave the field as it is.</td>
      
   </tr>
   </table>



2. After providing all of the required values in the above table, check the ***Terms & Conditions*** checkbox and click on the **Purchase button**.

   <img src="images/CustomDeployAzure3.png"></a> 

   >The deployment in Azure can take up to 30 minutes. At the end of the deployment, SonarQube will be configured to run as a Windows Service on the SonarQube VM. When the SonarQube service starts for the first time, it will configure its database. This can take an additional 15 minutes  to complete during which time the Azure deployment shows as completed but you still won't be able to reach the SonarQube home page. Please give SonarQube some time to update. Click <a href="https://blogs.msdn.microsoft.com/visualstudioalmrangers/2016/10/06/easily-deploy-sonarqube-server-in-azure/">here</a> for more information.


3. Once the deployment is successful, you will see the resources in Azure Portal. 

   <img src="images/azure_resources.png">

## Exercise 1: Create a SonarQube Project and configure Quality Gate

1. Access the **SonarQube** portal providing the DNS name suffixed by the port number. 

   >The default port for SonarQube is 9000. Copy the DNS name from the created Virtual Machine in Azure Portal as shown and append :9000 at the end. The final **SonarQube** URL will be **http://YOUR_DNS_NAME:9000**

   <img src="images/dns_name.png">

2. Open a browser and login to the SonarQube Portal using the following credentials-    
   >**Username= admin, Password= admin**

   <img src="images/sonarqube_portal.png">

3. Click on **Administration** in the toolbar, go to **Projects** tab and click **Management**. 

   <img src="images/sonar_admin.png">

4. Create a project with **Name** and **Key** as **MyShuttle**. 

   - **Name**: Name of the SonarQube project that will be displayed on the web interface.

   - **Branch**[Optional]: Code branch which is tracked and reviewed for quality of code.

   - **Key**: The SonarQube project key that is unique for each project.

   <img src="images/project_creation.png">


   Let us create a Quality Gate to enforce a policy which fails the gate if there are bugs in the code. A Quality Gate is a PASS/FAIL check on a code quality that must be enforced before releasing software. 

5. Click on **Quality Gates** menu and click **Create** in the Quality Gates screen.

   <img src="images/qualitygate.png">
   <br/>

   <img src="images/qg-create.png">

6. Enter name for the Quality Gate and click **Create**.

   <img src="images/qgcreate-popup.png">

7. Let us add a condition to check for the number of bugs in the code. Click on **Add Condition** drop down and select the value **Bugs**.

   <img src="images/qg-bugs.png">

8. Change the **Operator** value to **is greater than** and the **ERROR** threshold value to **0** (zero) and click on the **Add** button.

   > This condition means that if the number of bugs in Sonar Analysis is greater than 0 (zero), then the quality gate will fail and this fails the VSTS build.

   <img src="images/qgbug-add.png">

9. To enforce this quality gate for **MyShuttle** project, click on **All** under **Projects** section and select the project checkbox.

   <img src="images/qg-selectproject.png">

## Exercise 2: Setting up the VSTS project

1. Use <a href="https://vstsdemogenerator.azurewebsites.net/?name=SonarQube&templateid=77364" target="_blank">VSTS Demo Data Generator</a> to provision a project on your VSTS account.

   ![](images/vstsdemogen.png)

2. Provide the **Project Name**, the **SonarQube URL** that was created previously and click on **Create Project**. Once the project is provisioned, click the URL to navigate.

   <img src="images/vsts_project_provisioning.png">
   


## Exercise 3: Modify the Build to Integrate with SonarQube

Now that the SonarQube server is running, we will modify VSTS build definition to integrate with SonarQube to analyze the java code provisioned by the VSTS Demo Generator system.

1. Go to **Builds** under **Build and Release** tab, edit the build definition **SonarQube**. This is a Java application and we are using Maven to build the code. The Maven task in VSTS includes OOB support for SonarQube. All you need to is to point the SonarQube endpoint that you just created.


2. Click on the **Maven** task and scroll down to the **Code Analysis** section. Configure the SonarQube settings as follows-

   <table width="100%">
   <thead>
      <tr>
         <th width="40%"><b>Parameter</b></th>
         <th><b>Value</b></th>
         <th><b>Notes</b></th>
      </tr>
   </thead>
   <tr>
      <td><b>SonarQube Project Name</b></td>
      <td>MyShuttle</td>
      <td>The name of the project in SonarQube</td>
   </tr>
   <tr>
      <td><b>SonarQube Project Key</b></td>
      <td>MyShuttle</td>
      <td>The unique key of the project in SonarQube</td>
   </tr>
   </table>

      >Note: Here, the SonarQube Project Name and SonarQube Project Key values are based on the values you provide in Exercise 1: Step 3.
   


   <img src="images/build_configure.png"> 

3. Save and queue the build.

   <img src="images/build_in_progress.png">

4. You will see that the build has failed since the associated  **SonarQube Quality Gate** has **failed**. The  count of bugs is also displayed under **SonarQube Analysis Report**.

   <img src="images/build_summary.png">  

5. Click on the **Detailed SonarQube Report** link in the build summary to open the project in SonarQube.

   <img src="images/analysis_report.png"> 

## Exercise 4: Analyze SonarQube Reports

The link will open the **MyShuttle** project in the SonarQube Dashboard.  Under ***Bugs and Vulnerabilities***, we can see a bug has been caught. 

  <img src="images/sonar_portal.png"> 


  The page has other metrics such as ***Code Smells***, ***Coverage***, ***Duplications*** and ***Size***. The following table briefly explains each of these terms.

   <table width="100%">
   <thead>
      <tr>
         <th width="50%"><b>Terms</b></th>
         <th><b>Description</b></th>
      </tr>
   </thead>
   <tr>
      <td><b>Bugs</b></td>
      <td>An issue that represents something wrong in the code. If this has not broken yet, it will, and probably at the worst possible moment. This needs to be fixed.</b></td>
   </tr>
   <tr>
      <td><b>Vulnerabilities</b> </td>
      <td>A security-related issue which represents a potential backdoor for attackers.</b></td>
   </tr>
    <tr>
      <td><b>Code Smells</b> </td>
      <td>A maintainability-related issue in the code. Leaving it as-is means that at best maintainers will have a harder time than they should making changes to the code. At worst, they'll be so confused by the state of the code that they'll introduce additional errors as they make changes.</b></td>
   </tr>
    <tr>
      <td><b>Coverage</b> </td>
      <td>To determine what proportion of your project's code is actually being tested by tests such as unit tests, code coverage is used. To guard effectively against bugs, these tests should exercise or 'cover' a large proportion of your code.</b></td>
   </tr>
       <tr>
      <td><b>Duplications</b> </td>
      <td>The duplications decoration shows which parts of the source code are duplicated.</b></td>
   </tr>
       <tr>
      <td><b>Size</b> </td>
      <td>Provides the count of lines of code within the project including the number of statements, Functions, Classes, Files and Directories</b></td>
   </tr>
   </table>
   <br/>

  > In this example, along with the bug count, a character **D** is displayed which is known as **Reliability Rating**. **D** indicates that there is **atleast 1 critical bug** in this code. For more information on Reliability Rating, click <a href="https://docs.sonarqube.org/display/SONAR/Metric+Definitions#MetricDefinitions-Reliability">here</a>.


1. Click on the **Bugs** count to see the details of the bug.

   <img src="images/sonar_portal.png"> 

   <br/>

   <img src="images/bug_details.png">

2. You will see the error in line number 28 of **LoginServlet.java** file as **Make "List" serializable or don't store it in the session**.

   <img src="images/bug_details_2.png">

3. The error is because the session attribute accepts only serialized objects. This can be fixed by explicitly casting the list object to serializable. Lets fix this bug -

   Go to below path to edit the file in **VSTS** code tab:-
   
   >src/main/java/com/microsoft/example/servlet/LoginServlet.java

   Make the following changes in the code as shown:

   - Go to line number **28** and replace the existing code with below snippet.

      >session.setAttribute("employeeList", (Serializable)fareList);

      <img src="images/code_edit.png">

   - Import the below package.

      >import java.io.Serializable;

      <img src="images/code_import.png">

4. Commit the changes.

5. Once the CI build completes, you will see the Quality Gate as **Passed** in the build summary along with a brief view of **Test Results**, **Code Coverage** and link to SonarQube Analysis Report. . 

   <img src="images/build_summary_bug_fix.png"> 

6. Go to SonarQube portal. You will see the bug count is **0**. 

   <img src="images/bug_fix_sonar_portal.png"> 


## Summary

With  the **SonarQube** extension for **Visual Studio Team Services**, you can setup a CI build and automate the measurement of many metrics such as technical debt, code semantics, testing coverage. etc.

## Feedback

Please let <a href="mailto:devopsdemos@microsoft.com" target="_blank" >us</a> know if you have any feedback on this lab.
