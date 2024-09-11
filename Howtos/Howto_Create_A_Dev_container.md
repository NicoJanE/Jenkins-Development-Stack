---
layout: default_c
RefPages:
 - howto_create_a_dev_container
--- 
<small> _This file is part of: Jenkins development Stack_ 
_Copyright (c) 2024 Nico Jan Eelhart_
_This source code is licensed under the MIT License found in the  'LICENSE.md' file in the root directory of this source tree._
</small>
<br>

# Introduction 
This Jenkins container consists of:
- A WSL container (Jenkins in Jenkins) with a related Docker Jenkins image, which includes:
    - Jenkins
    - Mailhog

Jenkins may display the following warning: *"Building on the built-in node can be a security issue. You should set up distributed builds. See the documentation."* **However**, since this setup is intended for local development environments and not for organizational use (where security risks are more prominent), these risks are less significant for local systems with only a few connected devices. Therefore, you can safely dismiss the message in this context. <br><br>

## 1. Create the WSL
To create the necessary WSL and assign it to the Docker image, follow these steps:
- The Ubuntu 24.04 is part of this repository it can be found at: **../Jenkins Development Stack/_#Installs_Required\wsl_ubuntu_x64\Ubuntu_2204.1.7.0_x64/install.tar.gz**
- Run the following command to import the WSL distribution:
<pre class="nje-cmd-one-line">wsl --import Ubuntu-docker-Jenkins ./wsl2-distro  ../_#Installs_Required\wsl_ubuntu_x64\Ubuntu_2204.1.7.0_x64/install.tar.gz </pre>

This command will create a WSL distribution in the folder **Jenkins-Service/wsl2-distro** with the name **Ubuntu-docker-Jenkins**.
- Ensure that this WSL distribution is connected  to your Docker setup (**Do this after you have executed paragraph 2**)
    - In Docker -> Settings -> Resource -> WSL integration
    - In the **'Enable integration with additional distros:'** section (if you don't see this option,  press: Refetch distros)
    - Select ***Ubuntu-docker-Jenkins*** 
    - Press Apply & Restart (You may need to restart the Docker container manually)
Note: You don’t need to start the WSL distribution yourself. Docker will automatically start it when needed. You can verify this by observing that the WSL is running when the Jenkins container is started, even though you haven’t manually started the WSL.
<br><br>

## 2 Create & configure the container
To create the Docker container:
- Navigate to the service folder: ***Jenkins-Service***
- Run the following command to create the container
<pre class="nje-cmd-one-line">docker-compose -f compose_jenkins.yml up -d --build --force-recreate </pre>

#### Expected results 
- A new container named 'jenkins-service\jenkins-img-1'should be present in Docker Desktop and should be running.
- The Jenkins files created during the installation of plugins and Build Tasks you create will reside in the folder: **Jenkins-Service\jenkins_home**. When reinstalling Jenkins and using the same folder (as specified in the Docker Compose file), it will reuse these files. It’s recommended to **back them up** from time to time!
- You can access Jenkins on the host (if you haven’t changed the port) by navigating to **[http://localhost:8081/](http://localhost:8081/)**


### 2.1 Initial Jenkins configuration
- In Docker Desktop, examine the start-up log of the container. Near the top, you should find a code that is required for the initial login.
- Start Jenkins by opening:[http://localhost:8081/](http://localhost:8081/) in your Browser
- When prompted, enter the login code from the start-up log.
- On the next page, select the option **'Install recommended plugins'**. This may take some time to complete.
- After the plugins are installed, create your own login ID and password. Once done, Jenkins is ready for use..

> *Remark:*{: style="color: Grey;font-size:13px; "}
> <small>Access the host web services fom the container <br></small>
> <small>Because we use a bridged network and attached WSL we can access the host with: **host.docker.internal** for example: </small>
> - <small>  ***curl -s http://host.docker.internal:4072**/* </small>


### 2.2 Use the  Local Email Service
The image also installs an email-like server that can be used to send local emails in case a **build task fails**. This provides a centralized location to review error notifications.

To configure it in Jenkins:
- In the Main Jenkins Window choose **Manage Jenkins** -> followed by **System**
- In the **Jenkins Location** section, set the URL to: **http://localhost:8081/** (yes it complains, but thats fine for our local service)
- In the **E-mail Notification** section (**not** 'Extended E-mail Notification'): 
    - Set **SMTP server** to:  **mailhog**
    - Press **Advanced**
    - Set  **SMTP Port** to: **1025**
- Check the box for **Test configuration by sending a test e-mail**.
    - Enter any e-mail address (e.g., Jenkins-err@local.com) 
    - Press **Test configuration**
    - In the host Open the link: **[http://localhost:8025/](http://localhost:8025/)** in your browser, and the message should appear there.
<br><br>

## 3 Sample Build task
Here are the configuration instructions for a simple sample **build task** to get you started and to verify that the setup is correct. The build task will call an existing web page on the host. For this example, use the following address (ensure that the link returns a valid header from your host):
>  http://host.docker.internal:4072

<br>

#### Follow these steps:
- In the Jenkins Dashboard, select **New Item** and enter a name, for example: 'Is Running - Test WebSite'
- Select item type: **Freestyle project** then click **OK**
- Add a description, such as: 'Description Is Running - Test WebSite'
- Go tot **Build Triggers** and check the box for **Build periodically**
    - In the Schedule box enter:
    <pre class="nje-cmd-one-line-sm-ident">   H 10-11 * * *</pre>
    <span class="nje-ident"></span>This means to run between 10 and 11 AM, and choose a suitable minute Indicated by the: H
    - Optional you could add at the top:
    <pre class="nje-cmd-one-line-sm-ident">  TZ=Europe/Amsterdam</pre>
    <span class="nje-ident"></span> Choose your own timezone
    - Press Apply
- Scroll down to **Build Steps**
    - From **Add build step** select: **Execute shell**
    - In the **Command box** Enter: 
    <pre class="nje-cmd-one-line-sm-ident">   curl -sI http://host.docker.internal:4072 | grep -i "HTTP/" | grep -i "200 OK" || exit 1</pre>
    <span class="nje-ident"></span> This checks if the web site is up by inspecting the header.
    - Apply -> Save
- Scroll down to the **Post-build Actions**
    - From the **Add post-build action** chose: E-mail Notification
    - In the Recipients add a fake e-mail address , i.e. Dev@local-home. (It really does not matter which Email all will end up in mailhog) 
    - Apply -> Save

<br>

#### Test the Build Task
After saving the action, you should be redirected to the 'Is Running - Test WebSite' page. If not, search for this page..
- Check that your website is running.
- Press the **Build Now** button.
- After a short time the **Build History** Should display a **green** checkmark with date and time to indicate a successful run of the task.
- Turn off the WebSite.
- Press the **Build Now** button again.
- After a short time the **Build History** Should display a **red** checkmark with date and time to indicating a failed run of the task.
- Visit the Mailhog website at: [http://localhost:8025/#] (http://localhost:8025/) and check that a new mail from **Dev@local-home** has arrived with the failure announcement of the build task.
