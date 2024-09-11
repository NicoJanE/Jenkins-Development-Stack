_This file is part of: Jenkins development Stack_
_Copyright (c) 2024 Nico Jan Eelhart_

_This source code is licensed under the MIT License found in the  'LICENSE.md' file in the root directory of this source tree._
<br>

# 1. What
This describes how to create and update a 'Jenkins Development Stack'  This image is based on/Has the following charactristrics: 
- The image 'jenkins/jenkins:lts' (Debian based)
- It use the Host bridge Network setup to enable communication with the host.
- Form the container the hos can be reached vias the **host.docker.internal** for example:
    - ***curl -s http://host.docker.internal:4072*** to contact the localhost on the host with port 4072
- It includes a simple Mail server to which mails can be send in case of a build task error in Jenkins, This will provide you an overview of errors
- It uses a Windows WSL image to enable communication with the host, Otherwise less secure setup would be required to communicate it the host via the web. When needed a SSH tunnel can be added for communication with the host.

Versions used:
- Docker version 27.2.0, build 3ab4256
- WSL version:WSL version: 2.2.4.0 (Kernel: : 5.15.153.1-2)


# 2. How it was/can be created.
1. Creating the WSL

> *Remark:*{: style="color: Grey;font-size:13px; "}
> <smallYour Note sure if this is really needed <br></small>

To create and assign the WSL follow these steps:
- The Ubuntu 24.04 is part of this repository it can be found: **../Jenkins Development Stack/_#Installs_Required\wsl_ubuntu_x64\Ubuntu_2204.1.7.0_x64/install.tar.gz**
- Run the following command to import the WSL distribution:
<pre class="nje-cmd-one-line">wsl --import Ubuntu-docker-Jenkins ./wsl2-distro  ../_#Installs_Required\wsl_ubuntu_x64\Ubuntu_2204.1.7.0_x64/install.tar.gz </pre>
This will create a WSL distribution in the folder **Jenkins-Service/wsl2-distro** with the name ***Ubuntu-docker-Jenkins***
- Ensure that this WSL distribution is attached to your Docker setup:
    - In Docker -> Settings -> Resource -> WSL integration
    - In the **'Enable integration with additional distros:'** section (if you don't see this option,  press: Refetch distros)
    - Select ***Ubuntu-docker-Jenkins*** 
    - Press Apply & Restart (You may need to restart the Docker container manually)
Note: You don’t need to start the WSL distribution yourself. Docker will automatically start it when needed. You can verify this by observing that the WSL is running when the Jenkins container is started, even though you haven’t manually started the WSL.

1. The ***Dockerfile_Jenkins*** file:
<pre class="nje-cmd-multi-line">
networks:
  nje-jek:                          # Define a network
    driver: bridge
    ipam:
      config:
        - subnet: 172.16.0.0/16     # Custom subnet for fixed IPs
        
        
volumes:
  nje-shared_container-vol:         # named volumes, for: sharing data between containers, persistent data
                                    # can be used in service volumes, like: nje-shared_container-vol:./workspace/shared 
                                    
services:
  jenkins-img:                      # Our service 
    build: 	    
      context: .
      dockerfile: Dockerfile_Jenkins 
    ports:                                 
      - "8081:8080"                         # host:cont 
      - "50000:5000"                        # host:cont 
    networks:                               # commented because we use Host network in service
      nje-jek:                              # Use this Network
        ipv4_address: 172.16.0.10           # Assign a fixed IP to the container
    volumes:
      - ../workspace:/workspace             # Bind mount to .Net 8.o workspace directory        
      - /var/run/docker.sock:/var/run/docker.sock   # Jenkins access to the Docker daemon on your host.
      - ./jenkins_home:/var/jenkins_home             # Persists Jenkins data (jobs, configurations)    
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false    # Disables the Jenkins setup wizard    
    command: ["java", "-jar", "/usr/share/jenkins/jenkins.war", "--httpPort=8080"]  # Run Jenkins (Java) Note port is internal    
</pre>

2. The Compose file:
<pre class="nje-cmd-multi-line">
networks:
  nje-jek:                          # Define a network
    driver: bridge
    ipam:
      config:
        - subnet: 172.16.0.0/16  # Custom subnet for fixed IPs
        
        
volumes:
  nje-shared_container-vol:         # named volumes, for: sharing data between containers, persistent data
                                    # can be used in service volumes, like: nje-shared_container-vol:./workspace/shared 
                                    
services:
  jenkins-img:                      # Our service 
    build: 	    
      context: .
      dockerfile: Dockerfile_Jenkins 
    ports:                                 
      - "8081:8080"                         # host:cont 
      - "50000:5000"                        # host:cont 
    networks:                               # commented because we use Host network in service
      nje-jek:                              # Use this Network
        ipv4_address: 172.16.0.10           # Assign a fixed IP to the container
    volumes:
      - ../workspace:/workspace             # Bind mount to .Net 8.o workspace directory        
      - /var/run/docker.sock:/var/run/docker.sock   # Jenkins access to the Docker daemon on your host.
      - ./jenkins_home:/var/jenkins_home             # Persists Jenkins data (jobs, configurations)    
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false    # Disables the Jenkins setup wizard    
    command: ["java", "-jar", "/usr/share/jenkins/jenkins.war", "--httpPort=8080"]  # Run Jenkins (Java) Note port is internal
    
    
    
# Create and start: docker-compose -f compose_jenkins.yml up -d --build --force-recreate 
# After Jenkins is up and running, access it via http://localhost:8081.
# To access a service on the host: curl -s http://host.docker.internal:4072  
# Data Jenkins stored in:  ..\Jenkins Development Stack\Jenkins-Service\jenkins_home

</pre>

