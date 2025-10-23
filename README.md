
# Jenkins <span style="color: #409EFF; font-size: 0.6em; font-style: italic;"> -  Docker Setup & Usage Guide</span>

## ℹ️ Introduction

This is a **Jenkins build service** container. It includes a local "mail server" (Mailhog) to list build failures. By default using our default *External Docker network* settings

 <details>  
  <summary class="clickable-summary">
  <span  class="summary-icon"></span> <!-- Square Symbol -->
  <b>What's in the external network?</b>
  </summary>
  
> It can be useful to know what containe, IPv4 addresses and ports are used in a network
For this we have a script that displays the information for you. it can be found in my **Powershelll-Utilities** repository [here](https://github.com/NicoJanE/Powershell-Utilities). Use the `docker-netw-info` directory to execute the script
</details>

## ⚡ Setup

- For details on creating and using this container, see: [Setup](https://nicojane.github.io/Jenkins-Development-Stack/Howtos/howto_create_a_dev_container.html)

### ⚡Quick Setup

- `docker network create --subnet=172.40.0.0/24 dev1-net`
- `docker-compose -f compose_jenkins.yml up -d --build --force-recreate`

<br>

<p align="center">
  <a href="https://nicojane.github.io/Docker-Template-Stacks-Home/">
    <img src="assets/images/DTSfooter.svg" alt="Docker Template Stacks" width="400" />
  </a>
  </p>
