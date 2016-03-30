---
layout: post
title:  "Easy UrbanCode Deploy with Vagrant (part 1)"
date:   2016-03-30 07:45:12 +0100
categories: urbancode vagrant
---
![header](/assets/vagrant_article_header.jpg)


This article describes my process of automatically creating deployment targets for an IBM UrbanCode Deploy environment.

All the sourcecode is freely available in my [GitHub repository](https://github.com/niklaushirt/UrbanCode_Vagrant){:target="_blank"}.

<br>


## Prerequisites
Vagrant must be [installed on your local machine](https://www.vagrantup.com/docs/getting-started/index.html){:target="_blank"}. 
It can be downloaded and installed from [here](https://www.vagrantup.com/downloads.html){:target="_blank"}.


## Building the bases

In order to optimise performance, in this first article we will first build a base box image containing some basic stuff:

* Java JRE

And optional (to speed-up VM creation):

* Docker engine
* Docker compose
* NodeJS
* NPM

### 1_ Create the Vagrant File


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




### 2_ Create the the Base Box

Create a directory for the project and copy the above code into a file called *Vagrant* and finally run *vagrant up*

![terminal build](/assets/vagrant_Base_build.jpg)

```shell
mkdir my_ucd_agent_base
cd my_ucd_agent_base
nano Vagrant
#Paste the File Content
mkdir vagrant_data
vagrant up
```

   

When done, we'll create a box image of the base

```shell
vagrant package --output base_agent.box
```  
   

---

# Part 2 of this article is [here](http://niklaushirt.github.io/urbancode/vagrant/2016/03/30/vagrant_ucd_part2.html) 









