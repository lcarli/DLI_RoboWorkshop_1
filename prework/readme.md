# Deep Learning Institute Robot Workshop Pre-Requiste Lab

In this lab, we'll walk workshop attendees through the necessary pre-requisite steps.  At the completion of this lab, attendees should have a properly prepared Virtual Machine running in Azure that has the NVidia GPUs, and lab assets needed for successful completion of the workshop.

---

<a name="prereqs"></a>

## Pre-Requisites

To complete this lab, you will need to have the following items:

1. A personal laptop running Windows, MAC OS X, or Linux and a Web Browser.

1. An active azure subscription that you are willing to use for the lab.  **BE AWARE** the lab has you create N-Series VMs in Azure.  These VMs are special because they include NVIDIA GPU support and provide an extremely powerful environment for deep learning projects.  However, they are expensive when compared to other less capable VMs.  It is recommended that you use a trial subscription, or a subscription provided to you at a live event for the lab steps.  This will help to ensure that you are not billed for excessive VM utilization.

1. Nodejs v6.9.4 or later. You can download the current version of Nodejs from [https://nodejs.org/en/download/](https://nodejs.org/en/download/)

1. The Azure CLI installed. We will cover the installation below, but you can also follow the instructions from the [Install the Azure CLI](https://docs.microsoft.com/en-us/azure/xplat-cli-install) page.

1. Git installed.  If you don't have git on your system, you can install it from [https://git-scm.com/downloads](https://git-scm.com/downloads).  If you install git for the first time, make sure to update it's global config with your  name and email (You can use your preferred name and email address):

    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "Your Email"
    ```

1. An SSH client.  This is built-in on Linux or MAC OS X machines.  For windows users [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) is a popular choice.  Or if you are on Windows 10, you may want to consider using [Bash on Ubuntu on Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about).

--- 

<a name="timing"></a>

## Lab Timing

This lab will likely take you 2-3 hours to complete.  A large portion of that time is in the [Copying the Virtual Hard Disk (VHD) for the Virtual Machine](#task7) task where you copy a pre-existing 100GB Virtual Hard Disk File (VHD) from our storage account to yours.  Typically this copy will take over an hour by itself.  As you plan your timing through the lab, be aware that you may have an hour or more of inactivity during the [Copying the Virtual Hard Disk (VHD) for the Virtual Machine](#task7) task while the copy is completing.  That is a great time to step away for dinner, or a conference call, or just to catch up on whatever show you've been bingeing lately.

That said, it is also possible that the VHD copy will take no time at all, this occurs when the storage account you create happens to be on the same hardware subsystem, or "Stamp", as our source storage account that is hosting the source VHD.  If that happens by co-incidence then the VHD copy is performed as a "shadow copy" and happens nearly instantaneously. There is no way to predict or specify what storage stamp your storage account will be created on, so this lucky co-incidence will only happen in a small percentage of cases.

---

<a name="tasks"></a>

## Tasks

In this lab, you will complete the following tasks:

1. [Understanding the Azure Resources in this Lab](#task1)
1. [Cloning the GitHub Repo and opening Commands.txt](#task2)
1. [Creating your Azure Subscription](#task3)
1. [Installing the Azure Command-Line Interface (Azure CLI)](#task4)
1. [Logging into and configuring your Azure Subscription via the Azure CLI](#task5)
1. [Creating the Azure Resource Group, Storage Account, and Container](#task6)
1. [Copying the Virtual Hard Disk (VHD) for the Virtual Machine](#task7)
1. [Creating the Virtual Machine using the Copied VHD](#task8)
1. [Configuring Auto-shutdown on the new VM](#task9)
1. [Connecting to your Virtual Machine using SSH and starting Digits](#task10)
1. [Staring and Testing Jupyter on your VM](#task11)
1. [Manually Shutting Down, Starting, or Restarting your VM](#task12)
1. [Deleting your lab resources from your subscription](#task13)

---

<a name="task1"></a>

## Understanding the Azure Resources in this Lab

![Deep Learning Workshop Architecture](images/DeepLearningWorkshopArchitecture.png)

In this workshop, you will be using an Azure Virtual Machine (VM) to complete your work. The Virtual Machine will use a copy of a pre-existing Virtual Hard Disk (VHD) that we have created for your use.  The pre-existing VHD has Ubuntu 16.0.4 LTS installed, along with all of the deep learning tools, frameworks, data sets and jupyter notebooks that you will need for the lab.  The purpose of this prework is to walk you step-by-step through the process of configuring your Azure Subscription with the resources necessary to complete the workshop.  This section is provided to help you better understand what those resources are in an effort to help you better understand the subsequent tasks, and the workshop overall.

### Azure Locations (Data Regions)

An Azure "Location", or "Region". Specifies the location of the Azure Data Center where you want your resources to be created.  The location that you use is very important in this lab because the N-Series Virtual Machines that have NVIDIA GPUs on them are only available in the "**East US**" or ("**eastus**") data center at this time.  This workshop requires the use of the NVIDIA GPUs on the N-Series VMs, so we will need to make sure to provision all of our resources in the "**eastus**" data center.  

> **Note**: The list of data centers where N-Series VMs are available will grow over time.  You can view the current data centers that support them on them "[Products available by region](https://azure.microsoft.com/en-us/regions/services/)" page.  If you choose to use a location other than "**eastus**", you will need to replace all occurrances of "**eastus**" in the instructions below with the name of the location you've chosen.

### Azure Resource Groups

A typical solution in Azure requires multiple resources.  For example, in this workshop, we will create an Azure Storage Account to store a Virtual Hard Disk (vhd) file, and then we'll create a Virtual Machine based on that VHD.  In addition, we'll have Network Interfaces, Virtual Networks, Networks Security Groups and more.  To keep all of the resources organized, we'll place them all in a single "**Resource Group**".  Resource Groups offer a number of benefits in Azure:

- You can create an Azure Resource Manager (ARM) template for all of the resources in your group to help automate the deployment of your resources.
- You can grant others access to your Resource Group without having to give them access to your entire subscription.
- When you no longer need the resources for a soulution, you can easily delete all of them at once by simply deleting the Resource Group that contains them.


### Azure Storage accounts

Azure Storage Accounts are a core Azure offering.  They support the storage of data in a variety of formats using a number of built-in services.  In this lab, we will be using Azure Storage Blob Containers to store the Virtual Hard Disk (VHD) blob for our Virtual Machine.  To accomplish this, we will need to first provision an Azure Storage account in our subscription, and ensure that it is in the "**eastus**" location.  Azure Storage Accounts are globally accessible and as such need a globally unique name.  We'll talk more about how these resources will be named later.

### Azure Virtual Machines

Azure Virtual Machines provide an extremely powerful and flexible way to run either Windows or Linux virtual machines in the cloud.  Virtual Machines use Virtual Hard Disks (vhd) files as their "disks".  In this workshop, we will be using a pre-provisioned Virtual Hard Disk image that has the following pre-installed:

- Ubuntu 16.04 LTS
- DIGITS
- Jupyter

You will copy that pre-existing vhd image over to a storage account in your own subscription, and then use an ARM template to create a Virtual Machine based on that image.

In addition to the vhd, your Virtual Machine will also need some additional resources.  These will all be created for you by the ARM template, but it's helpful to understand them.

- A Network Interface (NIC) - This is the virtual version of the Network Interface (think ethernet port) on your computer.
- A public IP Address - This will be assigned to your NIC.  Be aware thought that this IP address will change each time your VM is deallocated and restarted.
- A Fully Qualified Domain Name (FQDN): this will be your vm name, followed by `eastus.cloudapp.azure.com` (assuming you create your vm in the `eastus` data center).  While the IP Address may change, the FQDN will always stay the same and is the preferred way to connect to your VM over the internet.
- A Virtual Network - This is the internal virtual network that our NIC is attached to.  We don't really need it for this workshop, but a virtual network would allow you to connect multiple virtual machines to it, and allow direct, private communication between those VMs on that network.
- A Network Security Group - These provide the sets of firewall access rules that Azure enforces for you.  The template you run will create a network security group that allows incoming access to ports 22 (for ssh), 80 (for jupyter) and 8888 (for DIGITS).  The template then assigns that Network Security Group to your NIC.

### Choosing a good "***&lt;name&gt;***"

In the diagram above, you see a number of resources with names that match the pattern "***&lt;name&gt;xxx***".  A number of the resources you create will need a globally unique name, so following the pattern is recommended.

You will want to replace the "***&lt;name&gt;***" place holder (including the ***&lt;*** and ***&gt;*** angle brackets) with something unique to you.  Suggestions include:

- Use only simple lower case alphanumeric (a-z,0-9) characters.  No underscores, dashes, or special symbols.
- Your first, middle and last initials and the current Month and Day in ***mmdd*** format.  For example, if your name was John Q. Doe (***jqd***) , and the current date was February 1st (***0201***), you might use "***jqd0201***" in place of the "***&lt;name&gt;***" place holder.  If so, the Resource Group name would be "**jqd0201group**", the storage account name would be "**jqd0201storage**", and the virtual machine name would be "**jqd0201vm**".  You get the idea.
- Any other short set of characters that will likely result in a globally unique name, and that makes sense to you.

Choose a good ***&lt;name&gt;*** now, and make a note of it so you can use it later in the lab.

Throughout the remainder of the documentation the syntax samples will use ***dli0201*** ("dli" being short for Deep Learning Institute and "0201" for February 1st) as a ***&lt;name&gt;*** place holder value.  **DO NOT USE dli0201** for yourself, as it would likely conflict with the resources created while documenting this lab, or with others who lazily use that same name. 

---

<a name="task2"></a>

## Cloning the GitHub Repo and opening Commands.txt

You will need to modify the majority of the commands below to use the ***&lt;name&gt;*** prefix you have chosen.  In addition, a number of the commands shown below are split across multiple lines for readability, but will need to be entered on a single line.  We have simplified this task for you by creating a "**[commands.txt](deploy/commands.txt)**" file in the github repo.  You can open that file, do some global search and replaces of your specific values, then simply copy and paste the commands from the file into your command prompt one by one as you work through the lab.  This will help keep things easy, reduce typos, and speed things up in general.

To do this, you will need to first clone the GitHub repo down to your personal workstation so you can get the file. 

1. From your system's command prompt or terminal, change to the folder where under which you want to clone the github repo (for example, your user's home directory):

    > **Note**: You can clone the repo on your workstation wherever you like, just make sure you know where it is!  We are just suggesting the home folder as a likely place.

    on Windows

    ```bash
    cd %HOMEPATH%
    ```

    in bash:

    ```bash
    cd ~
    ```

1. then use the git command line tool to clone the github repo:

    ```bash
    git clone http://github.com/dxcamps/DLI_RoboWorkshop_1
    ```

1. Once the github repo has been cloned, change into the `./DLI_RoboWorkshop_1/prework/deploy` folder

    ```bash
    cd ./DLI_RoboWorkshop_1/prework/deploy
    ```

1. You can list the contents of the folder with an `ls` (Mac or Linux) or `dir` (windows) command, and see the files in the folder:

    ```bash
    $ ls
    commands.txt  DeploymentHelper.cs  deploy.ps1  parameters.json
    deployer.rb   deploy-preview.sh    deploy.sh   template.json
    ```

1. Open the "**[commands.txt](deploy/commands.txt)**" file in the text editor of your choice.  You should see a number of place holders, like `<name>`, `<key1>`, `<publicip>` and `<fqdn>`.  We will be replacing each of those place holders with actual values as we work through the lab.  For now, the only one we know is the `<name>` prefix we chose in the last task.

1. In the text editor of your choice, do a global search and replace of the `<name>` place holder with the name prefix you chose above (our example, `dli0201`).

    - As and example, on Windows (use similar steps in your text editor if it is something other than notepad):

        - Open the "**[commands.txt](deploy/commands.txt)**" in Notepad, then from the edit menu, select "**Edit**" | "**Replace...**" from the menu bar, and in the "**Replace**" window, replace `<name>` with your prefix , for example `dli0201`.  Click the "**Replace All**" button, then close the "**Replace** window:

            ![Notepad Search And Replace](images/02010-NotepadSearchReplace.jpg)

            You should notice that all the occurrances of `<name>` have been replaced with your prefix (again, for example `dli0201`):

            ![Notepad Search And Replace Done](images/02020-NotepadSearchReplaceDone.jpg)

        - ***Keep your text editor open so you can easily copy the commands from it.***

1. The "**[commands.txt](deploy/commands.txt)**" file has now been updated  with the `<name>` place holders replaced with your prefix name.  ***You can use those updated commands as you work through this lab, rather than copying them from this document, and modifying them one at a time.  Also, the commands have all been properly formatted on a single line and should work on both a Windows "Command Prompt" (not PowerShell though) as well as at a Bash prompt on Windows, Linux or Mac.***

---

<a name="task3"></a>

## Creating your Azure Subscription

If you are attending a sponsored workshop you should be receiving instructions prior to the event on how to sign up for you Azure subscription or have been given credentials to access an existing subscription.  Follow the instructions in that email to create your subscription.

If you are running this prework before the workshop, make sure that it isn't so far in advance of the workshop that the pass may expire before the event date.  Also make sure to shutdown and deallocate (but not delete) the the virtual machine when you are done with the prework so that it doesn't consume the credits availalbe in your subscription.  We will show you how to both automatically (setup by default) as well as manually shutdown and deallocate your vm at the end of this lab.

If you do NOT have access to an Azure Pass, you can create a free Azure Trial subscription at [azure.com/free](http://azure.com/free).  You may have issues provising an NC6 VM on a free trial based on some initial quota restrictions placed on the Free Trial subscriptions, however you can [contact support through the Azure Portal](https://docs.microsoft.com/en-us/azure/azure-supportability/how-to-create-azure-support-request) and request an increase in the number of cores (up to at least 6) as well as access to the "Standard_NC6" VM sku for your subscription.

> **Note**: The free Azure trial currently grants you a US$200 credit for a period of 30 days.  If either the 30 day limit, or $200 credit is exceeded, your trial resources (except those with free usage) will be shut down.  You will need a credit card to sign up for the trial, but it is used for identification purposes only and you will not be charged UNLESS you choose to go on a pay-as-you go basis.

Lastly, if you choose to use a personal subscription other than one created by an Azure Pass or Free Trial, be aware that you will be responsible for the usage incurred during this lab.  For more information on the costs associated with Azure resources, see the [Azure pricing](https://azure.microsoft.com/en-us/pricing/) page.

---

<a name="task4"></a>

## Installing the Azure Command-Line Interface (Azure CLI)

The Azure CLI is a cross platform command line interface that you can use to manage resources in your Azure Subscription from your local workstation.  The Azure CLI is a Nodejs application and as such runs on Windows, Linux and Mac OS X. It's for that reason specifically that we have chosen it as the toolset for this lab.

1. Ensure that you have Nodejs v6.9.4 or later installed.  If you don't you can install it from [https://nodejs.org/en/download/](https://nodejs.org/en/download/).

1. From your system's command prompt or terminal window, issue the following command:

    > **Note**: You can alternatively use an installer if you are on Windows or Mac OS X.  To find out more about using an installer, view [Option 2: Use an installer](https://docs.microsoft.com/en-us/azure/xplat-cli-install#option-2-use-an-installer) on the [Install the Azure CLI](https://docs.microsoft.com/en-us/azure/xplat-cli-install) page in the Azure documentation.

    ```bash
    npm i -g azure-cli
    ```
    On OS X and Linux systems, you may need to use `sudo` before the command.

    ```bash
    sudo npm i -g azure-cli
    ```
---

<a name="task5"></a>

## Logging into and configuring your Azure Subscription via the Azure CLI

In this task, you'll login to your Azure subscription from your workstation's command line.  There are multiple ways to login to the Azure-CLI.  You can learn about alternate login methods here: [Log in to Azure from the Azure CLI](https://docs.microsoft.com/en-us/azure/xplat-cli-connect). We'll be using the "**Azure login with interactive login**" method:

1. From your system's command prompt or terminal, enter the following command to set the azure-cli into "Azure Resource Manager" mode (ARM):

    > **Note**: The first time you use the azure-cli you will likely see a prompt that starts with "`Microsoft Azure CLI would like to collect data about how users use CLI
commands and some problems they encounter...`", asking you to participate in azure-cli data collection.  You can accept or decline as you wish, it won't impact how the cli functions. 

    ```bash
    azure config mode arm
    ```

1. Next, to login to your Azure Subscription from the azure-cli, issue:

    ```bash
    azure login
    ```

1. The output should be something similar to the following (output shown wrapped for readability):

    ```bash
    info:    Executing command login
    info:    To sign in, use a web browser to open the page https://aka.ms/devicelogin 
             and enter the code XXXXXXXXX to authenticate.
    ```
1. As instructed, copy the `XXXXXXXXX` code from the info message, open your browser, and navigate to the [http://aka.ms/devicelogin](http://aka.ms/devicelogin) site to authenticate:

    - Paste the `XXXXXXXXX` code copied from above into the box, then click "**Continue**":

        ![Device Login Code](images/01010-DeviceLoginCode.png)

    - Then authenticate with the credentials for your Azure Subscription

        > **Note**: If you were given a pre-provisioned set of credentials to use for the workshop, make sure to use those here so that you sign into the proper Azure subscription.

        ![Azure Credentials](images/01020-AzureCredentials.png)

    - Back at the command prompt, you should get a login confirmation similar to the following:

        ```bash
        info:    Added subscription <Your Subscription Name>
        info:    Setting subscription "<Your Subscription Name>" as default
        info:    login command OK
        ```

1. If you only have a single azure subscription associated with the credentials you used above, then that subscription should be the current subscription.  However, if you have more than one subscription or if you want to just ensure the proper subscription is being used, follow these steps:

    - From the command prompt, or terminal window enter the following azure-cli command to get a list of the susbscriptions associated with your login:

        ```bash
        azure account list
        ```

    - You should get a result similar to the following:

        ```bash
        $ azure account list
        info:    Executing command account list
        data:    Name         Id                                    Current   State
        data:    ----------   ------------------------------------  --------  -------
        data:    Your Sub     wwwwwwww-wwww-wwww-wwww-wwwwwwwwwwww  true      Enabled
        data:    Azure Pass   zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz  false     Enabled
        info:    account list command OK
        ```

    - If more than one subscription is listed above, you need to ensure that the subscription you wish to be used is set as the "Current" subscription.  To do so, use the following azure-cli command:

        ```bash
        azure account set <Your Subscription Name or Subscription Id>
        ```

    - For example, if you are at a sponsored event and were given an Azure Pass to use, notice the subscription name of "`Azure Pass`" for the Azure Pass subscription.  We could then use this command to ensure that the "`Azure Pass`" subscription was the current subscription used by the azure-cli:

        > **Note**: If your subscription name has spaces in it, make sure to surround it in quotes.

        ```bash
        azure account set "Azure Pass"
        ```
    - Finally, to get the details of the current subscription, you can enter the following command:

        ```bash
        azure account show
        ```

    - The `azure account show` returns results similar to:

        ```bash
        info:    Executing command account show
        data:    Name                        : Azure Pass
        data:    ID                          : zzz...zzz
        data:    State                       : Enabled
        data:    Tenant ID                   : qqq...qqq
        data:    Is Default                  : true
        data:    Environment                 : AzureCloud
        data:    Has Certificate             : No
        data:    Has Access Token            : Yes
        data:    User name                   : dli2017001@outlook.com
        ```

1. We will be using a number of Azure "**Resource Providers**" in this lab.  Resource Providers enable us to manage specific types of resources in the Azure Platform via the "**Azure Resource Manager**" API that is used by the Azure-cli.  We need to enable, or "**Register**" each of the providers we will be using before we can use them.  The "**Resource Providers**" we need include:

    - "Microsoft.Storage"
    - "Microsoft.Network"
    - "Microsoft.Compute"
    - "Microsoft.DevTestLab"

1. We'll use the Azure-cli to register each one.  Use the following command to register the "**Microsoft.Storage**" resource provider:

    > **Note**: Occasionally the Azure-CLI will timeout when running a command.  This can be caused by a number of external factors.  The cool thing is you can simply re-run the statements if that occurrs.  Even if they succeeded the commands will simply ensure that the desired outcome was achieved.  If you receive timeout errors when running the statements below (or other statements later in the chapter), simply run the statements again.  If the errors are in regards to something else, do your best to understand and resolve the error and try again.

    ```bash
    azure provider register Microsoft.Storage
    ```

    Example output:

    ```bash
    info:    Executing command provider register
    + Registering provider Microsoft.Storage with subscription de9618cf-ca76-4a42-acf0-7f5f992a674a
    info:    provider register command OK
    ```

1. Repeat the `azure provider register` command for each of the other resource providers:

    ```bash
    azure provider register Microsoft.Network

    azure provider register Microsoft.Compute

    azure provider register Microsoft.DevTestLab
    ```

---

<a name="task6"></a>

## Creating the Azure Resource Group, Storage Account, and Container

The Azure Virtual Machine that you will be using for this lab will be based on a copy of a pre-existing Virtual Hard Disk (VHD) image that we have created for your use.  The pre-existing VHD image has Ubuntu 16.0.4 LTS installed, along with all of the deep learning tools, frameworks, data sets and jupyter notebooks that you will need for the lab.  In this task, you will create an Azure Resource Group, Storage Account, and Blob Container that will hold your copy of the VHD image.

1. First, to create your Resource Group, from your system's command prompt or terminal window, issue the following command. Make sure to replace the ***&lt;name&gt;*** place holder in the `<name>group` name with the naming prefix you chose above.

    > **Note**: Recall that we need to create all of our resources in the same "Location" and that because the N-Series NVIDIA backed GPU Virtual Machines are currently only availalbe in the "**eastus**" region, we want to sepcify that as the location for our storage account.

    > **Note**: Remember also that all the commands, with the proper values for the `<name>` and other place holders can be easily copied from the "**[commands.txt](deploy/commands.txt)**" file and pasted into your command prompt.  This will save you having to type each command, replace the place holders, etc..  For example, the following `azure group create` command with the `<name>` place holder value replaced with your prefix can be found at line 140 in the "**[commands.txt](deploy/commands.txt)**" file.  

    ```bash
    azure group create <name>group --location "eastus"
    ```

    The output should look something similar to:

    ```bash
    info:    Executing command group create
    + Getting resource group dli0201group
    + Creating resource group dli0201group
    info:    Created resource group dli0201group
    data:    Id:                  /subscriptions/zzz...zzz/resourceGroups/dli0201group
    data:    Name:                dli0201group
    data:    Location:            eastus
    data:    Provisioning State:  Succeeded
    data:    Tags: null
    data:
    info:    group create command OK
    ```

1. Next, create the Azure Storage Account within your new resource group, making sure again to use the appropriate location.  In the following command replace `<name>group` and `<name>storage` with the resource group and storage account names determined above:

    > **Note**: This command may take thirty seconds or longer to execute

    ```bash
    azure storage account create <name>storage --resource-group <name>group --location "eastus" --kind Storage --sku-name LRS
    ```
    sample output:

    ```bash
    info:    Executing command storage account create
    + Checking availability of the storage account name
    + Creating storage account
    info:    storage account create command OK
    ```

1. Now that you have a storage account, we need to retrieve the key needed to manage it:

    ```bash
    azure storage account keys list <name>storage --resource-group <name>group
    ```

    sample output:

    ```bash
    info:    Executing command storage account keys list
    + Getting storage account keys
    data:    Name  Key                                                                                       Permissions
    data:    ----  ----------------------------------------------------------------------------------------  -----------
    data:    key1  xxx...xxx==  Full
    data:    key2  yyy...yyy==  Full
    info:    storage account keys list command OK
    ```


1. From the output, copy the value of the "key1" key (shown as "`xxx......xxx==`" above), and replace the `<key1>` place holders in "**[commands.txt](deploy/commands.txt)**" with it (example shown in Notepad, use the editor of your choice).

    ![Replace Key1 in Commands.txt with Storage Key](images/06010-SaveStorageKeyToCommandsTxt.png)

1. Next, we'll create the container to store the vhd for our virtual machine:

    > **Note**: This is a SINGLE command wrapped across multiple lines for readability. You need to copy, or type the syntax below as a single line, with the appropriate values for the `<name>storage`  and  `<key1>` value placeholders.  It would be easier to just copy the command with your values in place from appx line 160 in "**[commands.txt](deploy/commands.txt)**".

    ```bash
    azure storage container create
      --account-name <name>storage
      --account-key <key1>
      --container vhds
    ```

    The output should resemble the following:

    ```bash
    info:    Executing command storage container create
    + Creating storage container vhds
    + Getting storage container information
    data:    {
    data:        name: 'vhds',
    data:        metadata: {},
    data:        etag: '"0x8D44AE4B4FB4B44"',
    data:        lastModified: 'Wed, 01 Feb 2017 20:55:51 GMT',
    data:        lease: { status: 'unlocked', state: 'available' },
    data:        requestId: '41ae1c0e-0001-00bb-41cd-7c1bea000000',
    data:        publicAccessLevel: 'Off'
    data:    }
    info:    storage container create command OK
    ```

---

<a name="task7"></a>

## Copying the Virtual Hard Disk (VHD) for the Virtual Machine

Now that we have the Azure Resource Group, Storage Account and Blob Container created, we can now copy the pre-existing VHD image provided for this workshop.


***COPYING THE PRE-EXISTING VHD FOR THE WORKSHOP TO YOUR STORAGE ACCOUNT COULD TAKE OVER AN HOUR.***

1. To copy the pre-existing VHD image from the hosted storage account into the container you just created, use the following command:

    > **Note**: This copy COULD take over an hour to complete.  However, it's also possible that it will complete almost immediately.  This could happen if the storage account you created just coincidentally happened to be on the same set of hardware that the source storage account exists on.  If that is the case, the copy is done as a "shadow" copy and takes no time at all.  In most cases though it has to transfer a 100GB VHD file to a separate set of disks and it just takes time for that to complete.

    > **Note**: The `--source-uri` value points to the location of the pre-created vhd image.  It points to a vhd blob named `msftnvidiaimage.vhd` that we are hosting for your to copy.  These are pre-existing values that need to be entered exactly as shown below.  Do not modify them.  This is the VHD image we have created for you that already has Ubnutu 16.04 LTS, Digits, and Jupyter pre-installed.  It is hosted by us in a storage container of our own.

    ```bash
    azure storage blob copy start 
        --dest-account-name <name>storage
        --dest-account-key xxx...xxx==
        --dest-container vhds 
        --source-uri https://dlirwsourcestorage.blob.core.windows.net/vhds/msftnvidiaimage.vhd
    ```

1. The command should return output similar to the following:

    ```bash
    info:    Executing command storage blob copy start
    - Start copying blob https://dlirwsourcestorage.blob.core.windows.net/vhds/msftnvidiaimage.vhd
    data:    Copy ID                               Status
    data:    ------------------------------------  -------
    data:    89fa6be5-33f8-4602-b97d-06d255e6a0c6  pending
    info:    storage blob copy start command OK
    ```

1. The copy could take up to 30-60 minutes or possibly longer (in tests it sometimes ran as long as 60 minutes), you can monitor the progress by repeatedly issuing the following command:

    > **Note**: The `msftnvidiaimage.vhd` blob name is the name of the pre-existing source vhd image being copied.  Do not change the name in the command below.

    ```bash
    azure storage blob copy show
      --account-name <name>storage
      --account-key <key1>
      --container vhds
      --blob msftnvidiaimage.vhd
    ```

    The output of the command above shows the copy status in the `Progress` and `Status` columns.  

    > **Note**: in the example shown below, the `Progress` value of `24528171008/107374182912` means that 24,528,171,008 bytes (22.8GB) of 107,374,182,912 (100GB) have been transferred, or in otherwords the copy is about 23% complete.  The `Status` column shows `pending` to indicate the copy is still in progress.

    ```bash
    info:    Executing command storage blob copy show
    + Getting storage blob information
    data:    Copy ID                               Progress                  Status
    data:    ------------------------------------  ------------------------  -------
    data:    89fa6be5-33f8-4602-b97d-06d255e6a0c6  24528171008/107374182912  pending
    info:    storage blob copy show command OK
    ```

1. Contiune to monitor the status of the copy by repeating the `azure storage blob copy show...` command above until the `Status` reads `success`:

    ```bash
    info:    Executing command storage blob copy show
    + Getting storage blob information
    data:    Copy ID                               Progress                   Status
    data:    ------------------------------------  -------------------------  ------
    data:    89fa6be5-33f8-4602-b97d-06d255e6a0c6  107374182912/107374182912  success
    info:    storage blob copy show command OK
    ```
---

<a name="task8"></a>

## Creating the Virtual Machine using the Copied VHD

We are almost ready, the final step is to deploy a new Virtual Machne (VM) to the resource group we created above that uses the VHD we just copied the base image for it's OS Disk.  Turns out that VMs need a bunch of other things as well including Network Interfaces, Virtual Networks, IP Addresses, Firewall Rules, etc.  Rather than have you create each one of these by hand and risk doing something wrong, we will use a pre-configured Azure Resource Manager (ARM) template.  The template file is in the same github repo as the other workshop content so first, we'll need to clone the repo, then we can change into the folder with the deployment script and run the azure-cli command to deploy the vm based on the template. 

1. From your system's command prompt or terminal, ensure that you are `preworklab/deploy` directory under the location where you cloned the `DLI_RoboWorkshop_1` repository to:

    ```bash
    cd ./DLI_RoboWorkshop_1/prework/deploy
    ```

1. As before, can list the contents of the folder with an `ls` (Mac or Linux) or `dir` (windows) command, and see the files in the folder:

    ```bash
    $ ls
    commands.txt  DeploymentHelper.cs  deploy.ps1  parameters.json
    deployer.rb   deploy-preview.sh    deploy.sh   template.json
    ```

1. The last two files, `parameters.json` and `template.json` are the ones we'll use in this task:

    > **Note**: You should take a couple of minutes to examine the `template.json` file to get a feel for what it will create.  However ***be careful to not make any changes to template.json***

    - `template.json` (**YOU DO NOT NEED TO MAKE ANY CHANGES IN THIS FILE**) contains the actual ARM template that defines all the resources that will be createad, e.g. The VM, Virtual Network, NIC, IP Address, Firewall Rules, etc.
    - `parameters.json` contains the values that are needed for the deployment, like the actual name of the VM, the location where it should be deployed, etc.  You'll edit this file and enter the value for your `<name>` prefix.

1. Open the `parameters.json` file in the text editor of your choice.  Verify that the location is correct.  Change the "***name_prefix***" parameter's value of `<name>` to the name prefix you chose above, and save your changes.

    Here's the original parameters.json file contents

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "location": {
                "value": "eastus"
            },
            "name_prefix": {
                "value": "<name>"
            }
        }
    }
    ```

    and with our sample ***dli0201*** value for ***&lt;name&gt;***:

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "location": {
                "value": "eastus"
            },
            "name_prefix": {
                "value": "dli0201"
            }
        }
    }

    ```


1. Use the Azure CLI to deploy the vm using the template (the complete command with placeholders replaced on on a single line is at line 221 in the "**[commands.txt](deploy/commands.txt)**" file:

    ```bash
    azure group deployment create
      --resource-group <name>group
      --name vmdeployment
      --template-file template.json
      --parameters-file parameters.json
    ```

    Once it completes, you should see output similar to the following:

    ```bash
    info:    Resource 'shutdown-computevm-dli0201vm' of type 'Microsoft.DevTestLab/schedules' provisioning status is Succeeded
    info:    Resource 'dli0201vm' of type 'Microsoft.Compute/virtualMachines' provisioning status is Succeeded
    info:    Resource 'dli0201nic' of type 'Microsoft.Network/networkInterfaces' provisioning status is Succeeded
    info:    Resource 'dli0201storage' of type 'Microsoft.Storage/storageAccounts' provisioning status is Succeeded
    info:    Resource 'dli0201ip' of type 'Microsoft.Network/publicIPAddresses' provisioning status is Succeeded
    info:    Resource 'dli0201vnet' of type 'Microsoft.Network/virtualNetworks' provisioning status is Succeeded
    info:    Resource 'dli0201nsg' of type 'Microsoft.Network/networkSecurityGroups' provisioning status is Succeeded
    data:    DeploymentName     : vmdeployment
    data:    ResourceGroupName  : dli0201group
    data:    ProvisioningState  : Succeeded
    data:    Timestamp          : 2017-02-02T00:09:48.480Z
    data:    Mode               : Incremental
    data:    CorrelationId      : 6be18319-1546-409b-8f2f-732923ad7571
    data:    DeploymentParameters :
    data:    Name         Type    Value
    data:    -----------  ------  -------
    data:    location     String  eastus
    data:    name_prefix  String  dli0201
    info:    group deployment create command OK
    ```

1. Finally, to connect to the vm, you will need to know it's fully qualified domin name (fqdn) and/or ip address.  You can get all the details about your vm using:

    > **Note**: When your VM is shutdown and "deallocated" it's IP address will be released.  It most likely will have a different address if you restart it again.  The FQDN (Fully Qualified Domain Name) however will remain the same, so in general the FQDN is an easier way to connect to your VM. 

    ```bash
    azure vm show --resource-group <name>group --name <name>vm
    ```

    sample output:

    ```bash
    info:    Executing command vm show
    + Looking up the VM "dli0201vm"
    + Looking up the NIC "dli0201nic"
    + Looking up the public ip "dli0201ip"
    data:    Id                              :/subscriptions/xxx...xxx/resourceGroups/dli0201group/providers/Microsoft.Compute/virtualMachines/dli0201vm
    data:    ProvisioningState               :Succeeded
    data:    Name                            :dli0201vm
    data:    Location                        :eastus
    data:    Type                            :Microsoft.Compute/virtualMachines
    data:
    data:    Hardware Profile:
    data:      Size                          :Standard_NC6
    data:
    data:    Storage Profile:
    data:
    data:      OS Disk:
    data:        OSType                      :Linux
    data:        Name                        :dli0201vm
    data:        Caching                     :ReadWrite
    data:        CreateOption                :Attach
    data:        Vhd:
    data:          Uri                       :https://dli0201storage.blob.core.windows.net/vhds/msftnvidia.vhd
    data:
    data:    Network Profile:
    data:      Network Interfaces:
    data:        Network Interface #1:
    data:          Primary                   :true
    data:          MAC Address               :00-0D-3A-12-95-E7
    data:          Provisioning State        :Succeeded
    data:          Name                      :dli0201nic
    data:          Location                  :eastus
    data:            Public IP address       :40.114.1.63
    data:            FQDN                    :dli0201vm.eastus.cloudapp.azure.com
    data:
    data:    Diagnostics Instance View:
    info:    vm show command OK
    ```

1. From the output, copy the "FQDN" (`dli0201vm.eastus.cloudapp.azure.com` above) value and replace the `<fqdn>` place holders in commands.txt.  Additionally, copy the "Public IP address" (`40.114.1.63` above) and replace the `<publicip>` placholders in the commands.txt.

    ![Update commands.txt with your fqdn and publicip](images/08010-SaveFqdnAndIp.png)

---

<a name="task9"></a>

## Configuring Auto-shutdown on the new VM

The VM Template we deployed has Auto-Shutdown enabled by default.  Unless you change it, your VM will shutdown automatically at 11:59pm Pacific Standard Time every night.  If you want to change that setting, or disable it, you can complete these steps.

1. Open the [Azure Portal](https://portal.azure.com) and login with the credentials for your Azure Subscription
1. Click on the "Virtual Machines" icon along the left of the portal, then select the `<name>vm` Virtual Machine to open it's overview blade.
1. From the list of settings along the left, click on "**Auto-shutdown**".  
1. Then in the "**Auto-shutdown**" blade, configure the option according to your desires.  You can read more about the feature here, [Announcing auto-shutdown for VMs using Azure Resource Manager](https://azure.microsoft.com/en-us/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/):

    > **Note**: By default, the ARM template we used configures your VM to shutdown (and deallocate it's resources to save hourly billing costs) at 11:59 PM Pacific Time.  It is recommended that you leave this as is so that you are guaranteed your VM will automatically shutdown at the end of the day to save you from being billed for un-needed compute time.  However, you may change these settings to fit your needs if you so desire.

    ![Auto Shutdown in Portal](images/09010-AutoShutdownInPortal.png)

---

<a name="task10"></a>

## Connecting to your Virtual Machine using SSH and starting Digits

1. The user name and password for the vm are:

    - user: `dliuser`
    - paasword: `Pwd@234567890`

1. Use the ssh client of your choice, and ssh into either the fqdn or ip address for your vm, and use the user name and password given above to login:

    ```bash
    ssh dliuser@<fqdn>
    Pwd@234567890
    ```

    or

    ```bash
    ssh dliuser@<publicip>
    Pwd@234567890
    ```

1. Once logged in, you should be in the `dliuser`'s `home` folder.  Next, clone the github repo for the workshop into the vm so you have a copy of all the code needed:

    ```bash
    cd ~
    git clone https://github.com/dxcamps/DLI_RoboWorkshop_1
    ```

    The repo should be cloned into:

    ```bash
    ~/DLI_RoboWorkshop_1
    ```

    or 

    ```bash
    /home/dliuser/DLI_RoboWorkshop_1
    ```

1.  Change into the `/usr/local/digits` folder

    ```bash
    cd /usr/local/digits
    ```

1. Run the `digits-devserver` server on port `8888`:

    > **Note**: the `&` at the end of the command allows starts the digits server in the background so you can cancel out of the command to return to the prompt if needed.

    ```bash
    ./digits-devserver --port 8888 &
    ```

1. Wait until the output shows something similar to:

    > **Note**: This could take up to two minutes or longer the first time you run it on your server.  Don't try to connect from your local browser until tyou see the `[INFO] Loaded x jobs.` message.

    ```bash
    [1] 3932
    ___ ___ ___ ___ _____ ___
    |   \_ _/ __|_ _|_   _/ __|
    | |) | | (_ || |  | | \__ \
    |___/___\___|___| |_| |___/ 5.0.0-rc.1

    Couldn't import dot_parser, loading of dot files will not be possible.
    2017-02-06 18:40:10 [INFO ] Loaded 7 jobs.
    ```

1. From your personal workstation, open a browser window and navigate to `http://<fqdn>:8888` where `<fqdn>` is the fully qualified domain name for your vm that you copied earier.  

    ```bash
    http://<fqdn>:8888
    ```

    You should see a page similar to the following load in your browser:

    ![Digits Server Home Page](images/DigitsServerHomePage.jpg)


---

<a name="task11"></a>

## Staring and Testing Jupyter on your VM

1. ***OPEN A SECOND SSH CONNECTION TO YOUR AZURE VM*** (keep the ssh session with DIGITS running in it open) to your vm (logging in again as `dliuser` with the password `Pwd@234567890`), and again change into the `~/DLI_RoboWorkshop_1/notebooks` folder.

    ```bash
    cd ~/DLI_RoboWorkshop_1/notebooks
    ```

1. Run the jupyter server on port 80 (***don't forget the `sudo`***):

    ```bash
    sudo jupyter notebook --port 80 &
    ```

1. Wait for output similar to:

    ```bash
    [1] 4043
    [I 18:42:41.404 NotebookApp] Writing notebook server cookie secret to /home/dliuser/.local/share/jupyter/runtime/notebook_cookie_secret
    [W 18:42:41.615 NotebookApp] WARNING: The notebook server is listening on all IP addresses and not using encryption. This is not recommended.
    [W 18:42:41.615 NotebookApp] WARNING: The notebook server is listening on all IP addresses and not using authentication. This is highly insecure and not recommended.
    [I 18:42:41.662 NotebookApp] Serving notebooks from local directory: /home/dliuser/DLI_RoboWorkshop_1/notebooks
    [I 18:42:41.662 NotebookApp] 0 active kernels
    [I 18:42:41.662 NotebookApp] The Jupyter Notebook is running at: http://[all ip addresses on your system]:80/
    [I 18:42:41.662 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
    ```

1. Back, on your personal laptop, open a browser and navigate to `http://<fqdn>:80` where `<fqdn>` is the fully qualified domain name of your vm that you copied earlier:

    ```bash
    http://<fqdn>:80
    ```

    You should see a page similar to the following:

    ![Jupyter Home Pageh](images/JupyterServerHomePage.jpg)

1. Click on the "**TestSetup.ipynb**" link to open the test notebook.

    ![Jupyter Test Notebook Link](images/JupyterTestNotebookLink.jpg)

1. From the Jupyter menu bar, select "**Cell**" | "**All Output**" | "**Clear**":

    ![Clear All Output](images/JuypyterClearAllOutput.jpg)

1. Once the output is cleared, you can run all the cells by selecting "**Cell**" | "**Run All**" from the menu bar:

    ![Run All](images/JupyterRunAll.jpg)

1. You should see output similar to the following:

    ![Run All Output](images/JupyterRunAllOutput.jpg)

1. The top section verifies that the NVIDIA GPU is present, and reports details on it:

    ![Verify GPU](images/JupyterVerifyGPU.jpg)

1. The rest of the Test page, loads a picture with some bottles in it, as well as the output of the training data that was generated for the image, then plots boxes around the bottles that the training process found in the image: 

    > **Note**: Notice the red boxes drawn around the three bottles found on the bathroom counter top.  It even outlined the partial reflection in of one of the bottles in the mirror:

    ![Jupter Bottle Bounding Boxes](images/JupyterBottleBoxes.jpg)

1. At this point, your VM is ready to use for the workshop.  You can return to the Juypter home page at `http://<fqdn>:80` and try out the "**GettingStarted/Getting Started with Deep Learning,ipynb**" and "**ObjectDetection/Object detection.ipynb**" notebooks.  Simply navigate to them to open them, and begin reading and following the instructions within.  It's ok if you don't complete them, they will be covered in the in-person workshop you attend. 

    ![Getting Started and Object Detection Notebooks](images/11010-OtherNotebooks.png)

---

<a name="task12"></a>

## Manually Shutting Down, Starting, or Restarting your VM

If your VM is not currently doing any processing and you don't need it for some time you should shut it down and deallocate it's resources so that you are not being billed for them.  Even if you are using an Azure Pass or Free Trial the VM will still be depleting your free credits if you leave it running for no reason.  Recall that in the [Configuring Auto-shutdown on the new VM](#task9) section we have already seen that by default your VM should be scheduled to automatically shutdown and deallocate it's resources at 11:59pm Pacific time every day.  However you can shut it down and deallocate it's resources manually at any time to even further reduce the impact it has on your bill or available free credits.

### Deallocating vs Shutting Down (Stopping) your VM

Your Azure Virtual Machine (VM) uses a number of resources like CPUs, memory, IP Addresses, etc..  It's those resources that you are being billed for while your VM is running.  As with any computer there are times when you wish to shut the VM down.  The question is, do you want to keep the resources the VM needs (CPUs, memory, IP Addresses) reserved while it is shutdown or not.  If you want to keep those resources reserved even when your VM is shutdown, you can "**stop**" your VM but keep the resources.  Be aware though that you will continue to be billed for those reserved resources even when your VM is shutdown.  The billing will be the same as if the VM was running.  For example, at the time this is being written, the NC6 VM that we we create in the "East US" region costs $0.90US/Hour.  You will be billed that price regardless if the VM is running or stopped if you have not deallocated the VMs resources

If, on the other hand, you know you don't need the VM back right away, you can instead "**deallocate**" those resources and release them back into the pool of resources that Azure can assign to other VMs. You will no longer be billed the hourly rate for the VM.  Just be aware that some resources, like your VM's IP Address, will be different when you start the VM back up again.  It's FQDN however will be mapped to the new IP Address so if you connect using the FQDN instead of the IP address everything should be fine.  In production, if having an IP Address change is a problem, you can reserve IP Addresses for your VMs.  Read [IP address types and allocation methods in Azure](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-ip-addresses-overview-arm) to learn more.

Lastly, it should be mentioned that there are other resources, like your VM's Virtual Hard Disk file (VHD) that must remain in place and be billed for even if your VM is deallocated.  The only way to completely stop billing for your VM is to delete it and all of the resources is uses.

### Manually Deallocating your VM to Stop Hourly Billing

1. Back on the workstation where you have installed the azure-cli and logged into your azure subscription, first get a list of the VMs you have:

    ```bash
    azure vm list
    ```

    Sample output:

    ```bash
    + Getting virtual machines
    data:    ResourceGroupName  Name        ProvisioningState  PowerState  Location  Size
    data:    -----------------  ----------  -----------------  ----------  --------  --------------
    data:    DLI0201GROUP       dli0201vm   Succeeded          VM running  eastus    Standard_NC6
    info:    vm list command OK
    ```

1. You can Deallocate your VM, release it's resource and stop it's hourly billing by issuing:

    ```bash
    azure vm deallocate --resource-group <name>group --name <name>vm
    ```

    Sample output for the dli0201vm:

    ```bash
    info:    Executing command vm deallocate
    + Looking up the VM "dli0201vm"
    + Deallocating the virtual machine "dli0201vm"
    info:    vm deallocate command OK
    ```

    An `azure vm list` would now show `VM deallocated` as the VM's `PowerState`:

    ```bash
    info:    Executing command vm list
    + Getting virtual machines
    data:    ResourceGroupName  Name        ProvisioningState  PowerState      Location  Size
    data:    -----------------  ----------  -----------------  --------------  --------  --------------
    data:    DLI0201GROUP       dli0201vm   Succeeded          VM deallocated  eastus    Standard_NC6
    info:    vm list command OK
    ```

### Other VM Management commands

You get the basic pattern from above, list your VMs to determine their state, then deallocate, start, stop, restart them as needed. Here are other commands you can use:

1. To Start your Azure vm:

    ```bash
    azure vm start --resource-group <name>group --name <name>vm
    ```

1. To Restart your Azure vm

    ```bash
    azure vm restart --resource-group <name>group --name <name>vm
    ```

1. To  Stop (but not deallocate) your VM

    > **Note**: As previously explained, stopping a VM will shut the VM down, but keep it's resources (like it's IP Address) reserved.  It will NOT stop the hourly billing for the VM.  If you want to stop the hourly billing you should instead **deallocate** your VM as described above.

    ```bash
    azure vm stop --resource-group <name>group --name <name>vm
    ```

1. You can also do all of the above within the Azure Portal.  If you get to the blade for your VM you can manage it's state from there:

    > **Note**: When you "**Stop**" a VM in the portal, you are actually "**Deallocating**" it. 

    ![Manage VM State in the Portal](images/12010-ManageVMStateInPortal.png)

---

<a name="task13"></a>

## Deleting your lab resources from your subscription

Once you are done with the lab and no longer need the resources (VM, VHD, etc) that you created during it, you can delete them from your subscription to completely stop any billing or free credit consumption for them.  

At the very beginning of the lab, we created a resource group, **`<name>group`** and every other resource we created was placed in that group.  The beauty of that is that now to delete all of the resources, we simply need to delete the group.  Super easy. 

***THIS IS NOT RECOVERABLE, IF YOU DELETE THE RESOURCE GROUP IT, AND ALL OF THE RESOURCES WITHIN, WILL BE DELETED. ONLY DELETE THE GROUP IF YOU KNOW YOU DO NOT NEED IT ANY MORE.  DO NOT DELETE THE GROUP PRIOR TO A WORKSHOP EVENT, YOU WILL NEED THE RESOURCES AT THE EVENT.  ONLY DELETE THE GROUP AFTER THE EVENT IS OVER AND ALL RESOURCES ARE NO LONGER NEEDED***

1. To delete the resource group, and all of the resources you created in this lab, you can simply run:

    ```bash
    azure group delete <name>group
    ```