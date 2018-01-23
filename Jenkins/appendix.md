# Installing Additional Software

## Installing Git Plugin

1. From the main page of the Jenkins portal, select **Manage Jenkins** and then select **Manage Plugins**

    ![Manage Jenkins](images/manage-jenkins2.png)

1. Select the **Available** tab. 

1. Enter **Git plugin** in the filter textbox 

1. Select **Git plugin** in the search list and select **Install without Restart**


## Installing Team Services Private agent

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