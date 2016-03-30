---
layout: post
title:  "Easy UrbanCode Deploy with Vagrant (part 1)"
date:   2016-03-30 07:45:12 +0100
categories: urbancode vagrant
---

This article describes my process of automatically creating deployment targets for an IBM UrbanCode Deploy environment.

## Prerequisites
Vagrant must be [installed on your local machine](https://www.vagrantup.com/docs/getting-started/index.html). 
It can be downloaded and installed from [here](https://docs.vagrantup.com/v2/getting-started/index.html).


## Building the bases

In order to optimise performance, in this first article we will first build a base box image containing some basic stuff:

* The IBM UrbanCode Deploy Agent installation files
* Java JRE
* Docker engine
* Docker compose
* NodeJS
* NPM

### Create the Vagrant File

First we start with the "config" section in which we define that the base image is based on Ubuntu 14.04.

```ruby
Vagrant.configure(2) do |config|
 
  config.vm.box = "ubuntu/trusty64"
```

For this article I chose to create an inline shell script in order to make it a simple on-file solution to start with.

```shell
 # Provision the Base image
  config.vm.provision "shell", inline: <<-SHELL
```

So first we update the package manager and install the unzip binary.

```shell
    #UPDATE    
    sudo apt-get update
    #GET UNZIP
    sudo apt-get install unzip
```

Next we install the Java JRE needed to run the UrbanCode Deploy Agent

```shell
  #INSTALL JAVA
    sudo apt-get install --fix-missing -y
    sudo apt-get install openjdk-6-jre-headless -y
```


#### Optional Steps

For my specific example, I need install the following additional elements:

* Docker engine
* Docker compose
* NodeJS
* NPM

Install the Docker Engine

```shell
   #INSTALL DOCKER
    sudo apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    sudo echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' > /etc/apt/sources.list.d/docker.list
    sudo apt-get update
    sudo apt-get install docker-engine -y
    sudo service docker stop
    sudo service docker start
    sudo usermod -aG docker vagrant
```



Install Docker Compose

```shell
     #INSTALL DOCKER COMPOSE
    curl -L https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m` > docker-compose
    sudo mv docker-compose /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
```


Install NodeJS and NPM

```shell
   #INSTALL NODE/NPM
    sudo apt-get install nodejs -y
    sudo apt-get install npm -y
```


And I pull some Docker images to speed up deployment

```shell
  #PULL DOCKER IMAGES
    docker pull node
    docker pull dockerui/dockerui
    docker pull tutum/haproxyXXX
```




### Create the the Base Box

Create a directory for the project and copy the above code into a file called Vagrant finally run the vagrant boxScreen
![My helpful screenshot]({{ site.url }}/assets/vagrant_Base_build.jpg)


```shell
mkdir my_ucd_agent_base
nano Vagrant
cd my_ucd_agent_base
mkdir vagrant_data
vagrant up
```

When done, we'll create a box image of the base

```
vagrant package --output base_agent.box
cd ..
mkdir my_ucd_agent
cd my_ucd_agent
cp ../my_ucd_agent_base/base_agent.box .
Building the IBM UrbanCode Deploy Agent Box
```

sdfadfs

```
# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure(2) do |config|
    # Use the base box for speed
    config.vm.box = "../1_BASE_BOX/package.box"
   
    config.vm.provider "virtualbox" do |vb|
        # Customize the amount of memory on the VM:
        vb.memory = "512"
    end
    
    (1..3).each do |i|    
        config.vm.define "agent#{i}" do |agent|
            # Create a private network, which allows host-only access to the machine using a specific IP.
            agent.vm.network "private_network", ip: "192.168.231.4#{i}"    #Creates agents in the 192.168.231.4x subnet
            
            # Share the agent installation file with the VM
            agent.vm.synced_folder "./ucd_config", "/ucd_config"
            agent.vm.provision :shell, :path => "./ucd_config/install.sh", :args => "'#{i}'"
        end
    end
end

```







