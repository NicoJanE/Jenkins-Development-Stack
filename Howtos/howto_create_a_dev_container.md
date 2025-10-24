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

# Jenkins <span style="color: #409EFF; font-size: 0.6em; font-style: italic;"> -  Docker Setup & Usage Guide</span>

## ℹ️ Introduction

This Jenkins container consists of:

- Jenkins
- Mailhog

Jenkins may display the following warning: *"Building on the built-in node can be a security issue. You should set up distributed builds. See the documentation."* **However**, since this setup is intended for local development environments and not for organizational use (where security risks are more prominent), these risks are less significant for local systems with only a few connected devices. Therefore, you can safely dismiss the message in this context. <br>

<details>  
  <summary class="clickable-summary">
  <span  class="summary-icon"></span> <!-- Square Symbol -->
  <b>What's in my network</b>
  </summary>
It can be useful to know what containe, IP4 addresses and ports are used in a network
For this I have a script that displays the information for you. it can be found in my **Powershelll-Utilities** repository [**here**](https://github.com/NicoJanE/Powershell-Utilities). Use the `docker-netw-info` directory to execute the scrip
</details>

## 🛠️ Create & configure the container

To create the Docker container:

- Make sure the external network exists (or create it)

<pre class="nje-cmd-one-line"> docker network create --subnet=172.40.0.0/24 dev1-net </pre>

- Navigate to the service folder: ***Jenkins-Service***
- Run the following command to create the container

<pre class="nje-cmd-one-line">docker-compose -f compose_jenkins.yml up -d --build --force-recreate </pre>

### 🔎 Verify Network

- Verify that both containers are on the same network: ``docker network inspect dev1-net``  
  Also the other containers in the network will be shown including details like Ip address
- Check the IP address inside a container: ``hostname -I``

### ✅ Expected results

- A new container named **'jenkins-service\jenkins-img-1**' should be present in Docker Desktop and should be running.
- Also a sub container **jenkins-service\mailhog-1**should be present.
- The Jenkins files created during the installation of plugins and Build Tasks you create will reside in the folder: **Jenkins-Service\jenkins_home**. When reinstalling Jenkins and using the same folder (as specified in the Docker Compose file), it will reuse these files. It’s recommended to **back them up** from time to time!
- You can access Jenkins on the host (if you haven’t changed the port) by navigating to **[http://localhost:8081/](http://localhost:8081/)**
<br><br>

### ⚙️ Initial Jenkins configuration

- In Docker Desktop, examine the start-up log of the container. Near the top, you should find a code that is required for the initial login.
- Start Jenkins by opening:[http://localhost:8081/](http://localhost:8081/) in your Browser
- When prompted, enter the login code from the start-up log.
- On the next page, select the option **'Install suggested plugins'**. This may take some time to complete.
- After the plugins are installed, create your own login ID and password. Once done, Jenkins is ready for use..

> *Remark:*{: style="color: Grey;font-size:13px; "}
> <small>Access the host web services fom the container <br></small>
> <small>Because we use a bridged network and attached WSL we can access the host with: **host.docker.internal** for example: </small>
> - <small>  ***curl -s http://host.docker.internal:4072**/* </small>

### 📮 How to  Use the Local Email Service

The image also installs an email-like server that can send local emails if a **build task fails**. This provides a centralized location for reviewing error notifications.

To configure it in Jenkins:

- In the Main Jenkins Window choose **Manage Jenkins** -> followed by **System**
- In the **Jenkins Location** section, set the URL to: **http://localhost:8081/** (yes it complains, but that's fine for our local service)
- In the **E-mail Notification** section (**not** 'Extended E-mail Notification'): 
  - Set **SMTP server** to:  **mailhog** (the docker service)
  - Press **Advanced**
  - Set  **SMTP Port** to: **1025** (from  docker compose)
- Check the box for **Test configuration by sending a test e-mail**.
  - Enter any e-mail address (e.g., Jenkins-err@local.com) 
  - Press **Test configuration**
  - In the host Open the link: **[http://localhost:8025/](http://localhost:8025/)** in your browser, and the message should appear there.

<br>

---

<details>
<summary class="clickable-summary">
  <span class="summary-icon"></span>
   <h2> Appendix I</h2><span style="color: #409EFF; font-size: 16px;; font-style: italic;"> -  Sample Jenkins Build task </span>
</summary>

## 📎 Sample Jenkins Build task

Here are the configuration instructions for a simple build task to help you get started and verify the setup. The task will call an existing web page on the host. For this example, use the following address, ensuring it returns a valid header from your host:
>http://host.docker.internal:4072

### Follow these steps to Create the Build

- In the Jenkins Dashboard, select **New Item** and enter a name, for example: 'Is Running - Test WebSite'
- Select item type: **Freestyle project** then click **OK**
- Add a description, such as: 'Description Is Running - Test WebSite'
- Go tot **Build Triggers** and check the box for **Build periodically**
  - In the Schedule box enter:
   <pre class="nje-cmd-one-line-sm-ident">   H 10-11 * * *</pre>
   <span class="nje-ident"></span>This means to run between 10 and 11 AM, and choose a suitable minute, indicated by the: H
  - Optional you could add at the top:
    <pre class="nje-cmd-one-line-sm-ident">  TZ=Europe/Amsterdam</pre>
    <span class="nje-ident"></span> Choose your own timezone
  - Press Apply
- Scroll down to **Build Steps**
  - From **Add build step** select: **Execute shell**
  - In the **Command box** Enter: 
    <pre class="nje-cmd-one-line-sm-ident">   curl -sI http://host.docker.internal:4002 | grep -i "HTTP/" | grep -i "200 OK" || exit 1</pre>
    <span class="nje-ident"></span> This checks if the web site is up by inspecting the header.
  - Apply
- Scroll down to the **Post-build Actions**
  - From the **Add post-build action** chose: E-mail Notification
  - In the Recipients add a fake e-mail address , i.e. Dev@local-home. (It really does not matter which Email all will end up in mailhog)
  - Apply -> Save

### Test the Build Task

After saving the action, you should be redirected to the 'Is Running - Test WebSite' page. If not, search for this page.

- Check that your website is running.
- Press the **Build Now** button.
- After a short time the **Build History** Should display a **green** checkmark with date and time to indicate a successful run of the task.
- Turn off the WebSite.
- Press the **Build Now** button again.
- After a short time the **Build History** Should display a **red** checkmark with date and time to indicating a failed run of the task.
- Visit the Mailhog website at: [http://localhost:8025/] (http://localhost:8025/) and check that a new mail from **Dev@local-home** has arrived with the failure announcement of the build task.

</details>

<br>
<div align="center"> ─── ✦ ───
</div>
