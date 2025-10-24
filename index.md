---
layout: default_c
RefPages:
 - howto_create_a_dev_container
--- 

<small>
_This file is part of: **Jenkins Development Stack**_
_Copyright (c) 2024 Nico Jan Eelhart_
_This source code is licensed under the MIT License found in the  'LICENSE.md' file in the root directory of this source tree._ </small>
<br><br>

# Jenkins <span style="color: #409EFF; font-size: 0.6em; font-style: italic;"> -  Docker Support Container</span>

## ℹ️ Introduction

This Docker image is designed to host Jenkins in a Debian-based container for local and remote Git projects. It also includes a limited **mail** server to list the failed **build tasks**. By default this container uses our default **External Docker network settings**, to make sure containers in this network can work together.

<details>  
  <summary class="clickable-summary">
  <span  class="summary-icon"></span> <!-- Square Symbol -->
  <b>What's in the external network?</b>
  </summary>
  
> It can be useful to know what container, IPv4 addresses and ports are used in a network
For this we have a script that displays the information for you. it can be found in my **PowerShell-Utilities** repository [here](https://github.com/NicoJanE/Powershell-Utilities). Use the `docker-netw-info` directory to execute the script.
</details>

## ⚡ Setup

While creating the container is straightforward, a few additional steps are necessary to ensure everything is set up correctly. There are no "quick setup" instructions available for this image. Please refer to the [**Setup**](./Howtos/howto_create_a_dev_container) document, which covers installation and configuration.
