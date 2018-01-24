# Working with Jenkins

[Jenkins](https://jenkins.io/) is a very popular Java-based open source continuous integration (CI) server that allows teams to continuously build applications across platforms.

Visual Studio Team Services (VSTS) includes Team Build, a native CI build server that allows compilation of applications on Windows, Linux and Mac platforms. However, it also integrates well with Jenkins for teams who already use or prefer to use Jenkins for CI.

There are two ways to integrate VSTS with Jenkins

* One way is to completely replace the VSTS Build with Jenkins. This involves the configuration of a CI pipeline in Jenkins and a web hook in VSTS that invokes the CI process when source code is pushed by any member to a repository or a branch. The VSTS Release Management will be configured to connect to the Jenkins server through the configured Service Endpoint to fetch the compiled artifacts for the deployment.

* The alternate way is to use Jenkins and Team Build together. In this approach, a Jenkins build will be nested within the VSTS build. A build definition will be configured in the VSTS with a **Jenkins** task to queue a job in Jenkins and download the artifacts produced by the job and publish it to the VSTS or any shared folder from where it can be picked by the Release Management. This approach has multiple benefits -

    1. End-to-end traceability from work item to source code to build and release
    1. Triggering of a Continuous Deployment (CD) when the build is completed successfully
    1. Execution of the build as part of the branching strategy

This lab covers both the approaches and the following tasks will be performed

* Provision Jenkins on Azure VM using an Azure Marketplace Jenkins Template
* Configure Jenkins to work with Maven and VSTS. 
* Create a build definition in Jenkins
* Configure VSTS to integrate with Jenkins
* Setup Release Management in VSTS to deploy artifacts from Jenkins

## Pre-requisites

1. Microsoft Azure Account: You will need a valid and active azure account for the lab.

1. You need a **VSTS** account and a [Personal Access Token (PAT)](https://docs.microsoft.com/en-us/vsts/accounts/use-personal-access-tokens-to-authenticate)

1. You will need [Putty](http://www.putty.org/), a free SSH and Telnet client

1. You will need the **Docker Integration** extension installed and enabled for your VSTS account. You can perform this step later while using the VSTS Demo Generator.

## Setting up the VSTS project

1. Use [VSTS Demo Generator](https://vstsdemogenerator.azurewebsites.net/?name=MyShuttleDocker&templateid=77373) to provision a team project on the VSTS account
    
    ![VSTS Demo Gen](images/vstsdemogen-1.png)

1. Select **MyShuttleDocker** for the template

    ![VSTS Demo Gen](images/vstsdemogen-2.png)

    > Using the VSTS Demo Generator link will automatically select the MuShuttleDocker template in the demo generator for the team project creation. To use other team project templates, use this alternate link:  https://vstsdemogenerator.azurewebsites.net/

## Setting up the Jenkins VM

1. To configure Jenkins, the Jenkins VM image available on the Azure MarketPlace will be used. This will install the latest stable Jenkins version on a Ubuntu Linux VM along with the tools and plugins configured to work with Azure.

    <a href="https://portal.azure.com/#create/azure-oss.jenkinsjenkins" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
    </a>

1. Once the Jenkins VM is provisioned, click on the **Connect** button and make a note of the \<username> and the \<ip address>. This information will be required toconnect to the Jenkins VM from ***Putty***

    ![SSH Connection Info](images/vmconnect_ssh1.png)

1. Jenkins, by default, listens on port 8080 using HTTP. To configure a secure HTTPS connection, an SSL certificate will be required. If HTTPS communication is not being configured, the best way to ensure the sign-in credentials are not leaked due to a "Man-in-the-middle" attack is by logging-in using the SSH tunneling. An SSH tunnel is an encrypted tunnel created through a SSH protocol connection, that can be used to transfer unencrypted traffic over an unsecured network. To initiate a SSH tunnel, the following command needs to be run from a Command Prompt.
    ````cmd
   putty.exe -ssh -L 8080:localhost:8080 <username>@<ip address>
    ````
    ![Connecting from Putty](images/ssh2.png)

   > To initiate the above command, the Putty.exe needs to be placed in the path selected in the Command Prompt or the absolute path of the Putty.exe need to be provided in the command. 


1. Login with the user name and password that was provided during the provisioning of the VM.

1. Once the connection is successful, open a browser on the host machine and navigate to the URL [http://localhost:8080](http://localhost:8080). The **Getting Started** page for Jenkins will be displayed. 

1. The initial password needs to be provided in the **Getting Started** screen to unlock Jenkins. For security reasons, Jenkins will generate a password and save it in a file on the server. 

   ![Jenkins Initial Password](images/jenkinsinitialemptypwd.png)

    > At the time of writing this lab, an open issue in Jenkins was noted where the setup wizard would not resume after restart, skipping some of the steps listed below. If you do not see the screen above, steps 5 to 7 will not work. The workaround is to use the default user name *admin* with the initial admin password (explained in step #7 below).

1. Return to the **Putty** terminal and type the following command to open the log file that contains the password. Copy the password
    >sudo vi /var/lib/jenkins/secrets/initialAdminPassword

    *You can double click on the line and use **CTRL+C** to copy the text and place it in the clipboard. Press **ESC and then :q!** to exit the vi editor without saving the file*

1. Return to the browser, paste the copied text and click the **Continue** button.

    ![Unlock Jenkins - First Time](images/jenkinsinitialpwd.png)

    > Jenkins has a vast ecosystem with a strong and active open source community users contributing hundreds of useful plugins. While configuring Jenkins, you can either install the most commonly used plugins or pick the plugins that you want.

1. The Maven plugin is also required but will be installed later. Click on the **Install suggested plugins** option to initiate the configuration.

    ![Customize Jenkins Plugins](images/customizejenkins-plugins.png)

1. A new *Admin* user needs to be created for Jenkins. Provide a user name and password and click on the **Continue** button.

    ![Create Admin User for Jenkins](images/firstadminuser.png)

1. Jenkins will now be ready for use. Click on the **Start using Jenkins** button to start using it.

    ![Jenkins Ready](images/jenkinsready.png)

## Installing and Configuring Maven

 > Starting with Jenkins version 2, Maven plugin is not installed by default. You will need to install this manually

1. Select the **Manage Jenkins** option on the main page of the Jenkins portal. This will take you to the Manage Jenkins page, the centralized one-stop-shop for all the Jenkins configuration. From this screen, you can configure the Jenkins server, install and upgrade plugins, keep track of system load, manage distributed build servers, and more!

1. Select the **Manage Plugins** option

    ![Manage Jenkins](images/manage-jenkins1.png)

1. Select the **Available** tab and search for **maven-plugin** in the filter box

1. Select the **Maven Integration Plugin** option and click on the **Install without restart** button to install the plugin. 

    ![Install Maven](images/installmavenplugin.png)

1. Once the plugin is installed, select the **Manage Jenkins** option and then select **Global Tool Configuration** option.

    ![Global Tool Configuration](images/manage-tools-config.png)

1. Jenkins provides out-of-the-box support for Maven. Since the Maven is not yet installed, it can be manually installed by extracting the ***tar*** file located in a shared folder. Alternatively, Jenkins can install Maven automatically. When the **Install automatically** option is selected, Jenkins will download and install Maven from the Apache website when a build job requires it, first time.

    We will install Maven version 3.5, the latest version at the time the lab is written

    ![Maven Installer](images/maveninstallerconfig.png)

1. Click on the **Apply** button and then click the **Back to Dashboard** option to return to the home page.

## Creating a new build job

1. From Jenkins home page, click on the **New Item** option. Provide a name for the build definition, and select the **Maven project** and click **OK** to save

   ![](images/newbuilddef.png)

1. Now scroll down to the **Source code Management** section. Select the **Git** option and provide the clone URL of the VSTS Git repo. It would be typically in the format **http://\<your account name>.visualstudio.com/
 \<your project name>/_git/MyShuttle**

   ![Configuring VSTS Git URL](images/jenkins-vstsrepo.png)

   **Note:** The VSTS Git repos are private and hence will need the user credentials to be provided to access the repository. If the Git credentials are not yet configured, it can be done from the VSTS. Select the **Clone** option, provide the user name and password and then click on the **OK** button.

   ![Generating Git Credentials](images/vsts-generategitcreds.png)

1. Select the **Add | Jenkins** option to add a new credential. provide the user name and password created in the previous step and click on the **Add** button to close the wizard

    ![Adding Credentials to Jenkins](images/jenkinscredentials.png)

1. Select the credential from the drop-down. The error message should disappear now

   ![VSTS Git config in Jenkins](images/jenksaddvstsgit.png)

1. The source code for the application has both unit tests and UI tests. For the lab, the unit tests will be included but the UI tests will be skipped.

1. Scroll down to the **Build** section. Provide the following text in the **Goals and Options** field

   >package -Dtest=FaresTest,SimpleTest

1. Click on the **Save** button to navigate to the main page of the project that was just created

   ![Build Settings in Jenkins](images/jenkins-buildsettings.png)

1. The final configuration in this lab is to add a *Post-build* action to publish the artifacts. Scroll down to the **Post-Build Actions**  section, click on the **Add post-build action** option and select the option **Archive the artifacts** from the list.

   ![Post Build Action](images/jenkinspostbuildaction.png)

1. In the **Files to archive** test box, provide **target/*.war** and click the **Save** button to complete the configuration and return to the project page

   ![Archive War](images/jenkinsarchiveartifacts.png)

1. Click on the **Build Now** option to start an adhoc build

1. The build progress will be displayed below the left side navigation menu

   ![Running Ad-hoc Build](images/adhocbuild.png)

1. To view the build details including the build artifacts namely the .war file, click on the build number.

   ![Build Details](images/builddetails.png)

   ![Build Artifacts](images/buildmodules.png)

1. Click on the **Test Results** option to view the results of the unit tests that were included in the build definition.

## Approach 1: Using Jenkins without the VSTS Build

In this section, the Jenkins will be configured to run separately. A service hook will be configured in VSTS to trigger a Jenkins build whenever source code changes are pushed to a particular branch.

1. To configure the service hook, navigate to the VSTS team project page, click on the Settings icon and choose the **Service Hooks** option

    ![Navigate to service hooks page](images/servicehooks.png)

1. On the **Service Hooks** screen, click on the **Create subscription** button

1. In the *New Service Hook Subscriptions* screen, click on the **Jenkins** option and then click the **Next** button.

   ![Create a new subscription](images/vsts-createjenkinsservice.png)

1. Select the **Code pushed** option for the **Trigger on this type of event**

1. Select the MyShuttleDocker as the **Repository**, master as **Branch** and then click on the **Next** button.

   ![VSTS - Trigger Code Pushed](images/vsts-jenkinssubscription1.png)

1. Provide the following details in the **Select and configure the action to perform** screen 
   1. In the **Perform this action** field, select the **Trigger generic build** option

   1. In the **Jenkins base URL** field, Provide the URL of the Jenkins server in the *http://{ip address or the host name}* format

   1. In the **User name** and **User API token (or password)** fields, provide the user name and password already configured for Jenkins

1. Click on the **Test** button to validate the configuration and then click on the **Finish** button to complete the subscription.

   ![VSTS - Jenkins Info](images/vsts-jenkinssubscription2.png)

The VSTS will now automatically notify Jenkins to initiate a new build whenever any source code changes are committed on the repository. 

## Approach 2: Wrapping Jenkins Job within the VSTS Build

In this section, Jenkins will be included as a job within a VSTS Team Build. The key benefit of this approach is the end-to-end traceability from the work items to the source code to the build and release provided by the VSTS.

To begin, an endpoint to the Jenkins Server for communication with VSTS will be configured.

1. In the **Admin | Services** section, click on the **New Service Endpoint | Jenkins** option to create a new endpoint.

1. In the **Add new Jenkins Connection** screen, provide a connection name, Jenkins server URL in the  *http://[server IP address or DNS name]* format, Jenkins user name and password. Click on the **Verify Connection** button to validate the configuration and then click on the **Ok** button.

   ![Jenkins Endpoint](images/jenkinsendpoint.png)

The next step would be to configure the build definition.

1. Click on the **Build and Release** hub, select the Builds section and click on the **+New** button to create a new build Definition

1. In the Choose a template window, select the out-of-the-box **Jenkins** template and click on the **Apply** button

    ![Jenkins Template](images/jenkinsbuildtemplate.png)

1. In the *Process* step, provide a name for the definition, select Hosted Linux Preview as the Agent Queue, provide **MyShuttle** as the Job name and then select the Jenkins service endpoint created earlier.

    ![Jenkins Settings in Team Build](images/vsts-buildjenkinssettings.png)

1.  Next, select the **Get Sources** step. Since Jenkins is being used for the build, there is no need to download the source code to the VSTS build agent. Enable the **Advanced settings** option and select the **Don't sync sources** option.

    ![Get Sources Settings in Team Build](images/vsts-getsourcessettings.png)

1. Next, select the **Queue Jenkins Job** step. This task queues the job on the Jenkins server. Make sure that the services endpoint and the job name are correct. The two options available at this step will be left unchanged. 

     ![Jenkins Settings in Team Build](images/vsts-buildjenkinssettings1.png)

>The **Capture console output and wait for completion** option, when selected, will capture the output of the Jenkins build console when the build runs. The build will wait until the Jenkins Job is completed. The **Capture pipeline output and wait for pipeline completion** option is very similar but applies to Jenkins pipelines (a build that has more than one job nested together). 

1. The **Jenkins Download Artifacts** task will download the build artifacts from the Jenkins job to the staging directory

    ![Download Jenkins Artifact](images/downloadjenkinsartifact.png)

1. The **Publish Artifact drop** will publish to the VSTS. 

1. Click on the **Save & queue** button to complete the build definition configuration and initiate a new build. 

## Deploying Jenkins Artifacts with Release Management

The next step is to configure the VSTS Release Management to fetch and deploy the artifacts produced by the build.

1. Since the deployment is being done to Azure, an endpoint to Azure will be configured. An endpoint to Jenkins server will also be configured, if not configured earlier.

1. After the endpoint creation, click on the **Build & Release** hub and then select the **Releases** section. Click on the **+ Create a new Release definition** button to initiate a new release definition

1. Since a web application will be published to Azure, the **Azure App Service Deployment** template will be used for the configuration.

   ![New Release Definition](images/newreleasedefintion.png)

1. The default environment for deployment will be named as **Dev**

   ![New Environment](images/rm_environment.png)

1. To link the release definition with the MyShuttle build on Jenkins, click on the **+Add** button to add an artifact

1. In the **Add artifact** screen, Select **Jenkins** as the *Source type*, select the Jenkins endpoint configured earlier and provide **MyShuttle** for the *Source(Job)*

   >The Source(Job) should map to the project name configured in Jenkins

1. If you have configured Jenkins server and the source correctly, you will get a message showing the output of the build, in this case it should be ***myshuttledev.war***

   ![Add Jenkins artifact](images/rm_addjenkinsartifact.png)

1. You are now ready to deploy!

1. You can refer to the [Deploying Tomcat+MySQL application to Azure with VSTS](../tomcat/) if you want to continue with the deployment.


## Logging into Jenkins with the default credentials

1. To log in to Jenkins when you did not get a chance to configure the initial admin, you can use the default user name **admin**

1. Copy and paste the password text from `\var\lib\jenkins\secrets\initialAdminPassword` 

1. To change password, click on the user name on the top-right 

1. Select **Configure**. Scroll down to the **password** section. Specify a new password and click **Save**


# Appendix

## Installing Git Plugin

1. From the main page of the Jenkins portal, select **Manage Jenkins** and then select **Manage Plugins**

    ![Manage Jenkins](images/manage-jenkins2.png)

1. Select the **Available** tab. 

1. Enter **Git plugin** in the filter textbox 

1. Select **Git plugin** in the search list and select **Install without Restart**


## Installing VSTS Private agent

1. From VSTS, select **Admin**|**Agent Queue**

1. Select **Download agent**

    ![Agent Queue](images/vsts-agentqueue.png)

1. If you are accessing this page from the VM, it should default to **Linux**. Otherwise, select the tab.

1. Click **Download** to start downloading the agent. This is typically saved in the *Downloads* folder

    ![Download VSTS agent](images/downloadvstsagent.png)

1. Open a terminal window and enter the following commands one-by-one

    ````cmd
    mkdir vstsagent
    cd vstsagent
    tar -zxvf ../Downloads/vsts-agent-linux-x64-2.126.0.tar.gz
    ````
1. Once the files are extracted, run `./config.sh` to configure the agent. You will need to enter the VSTS URL and provide your PAT

1. After you have configured, start the agent by running the following command `./run.sh`