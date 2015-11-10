# Docker Lucee Cluster using docker-compose

This repository is a seed project to get [Lucee](http://lucee.org/) running as a cluster behind [NGINX](https://www.nginx.com/) acting as a loadbalancer within docker containers.

The Lucee Docker image used is available on Docker Hub: https://hub.docker.com/u/lucee/

### Big Thanks to Ramki Krishnnan
This repo is heavily based on this blog post: http://www.ramkitech.com/2015/10/docker-tomcat-clustering.html His videos and posts on Tomcat clustering and loadbalancing on NGINX were very helpful as well.

### Lucee Base Images
Lucee provides a number of different base images for your Lucee Docker containers to run your project. This project aims to use `docker-machine` and `docker-compose` to manage the docker containers as an alternative to the very helpful project by [Geoff Bowers](https://github.com/modius) which combines Vagrant/Docker: [Lucee Docker Workbench](https://github.com/modius/lucee-docker-workbench).

The above workbench was very helpful in getting me started with Docker, but I wanted to use `docker-compose` to start up my app locally, on VirtualBox, using `docker-machine`. The `yaml` configuration surrounding docker-compose was easiers for me to grasp than the vagrant approach, but I'm sure both have their merits. I also want to better understand Geoff's [tutum-prod.yml](https://github.com/modius/lucee-docker-workbench/blob/master/tutum-prod.yml) yaml files a bit more as well since [Tutum](http://tutum.co/) provides the ability to scale your services. This uses an almost nearly identical syntax to `docker-compose`. I also want/need to better understand how to handle environment level configuration / overrides to really make this a useful workflow.

### Pre-requisites

* Install DockerToolbox
* Ensure a docker machine is created and running.
* Be sure your docker-machine is active and that you've set the docker machine environment variables by following the instructions after running the `docker-machine env machine-name-here` command.
* You can check the docker-machine is active by running the command `docker-machine active` and it will output the name of the active docker-machine
* `docker-machine ip machine-name-here` will give you the IP your `lb` service is running on.

### The `docker-compose.yml` file
```yaml
lb:
  image: nginx
  ports:
   - "80:80"
  volumes:
   - ./cluster/default.conf:/etc/nginx/conf.d/default.conf
  links:
   - lucee1:lucee1
   - lucee2:lucee2
   - lucee3:lucee3
lucee1:
  image: lucee/lucee4:latest
  volumes:
   - ./cluster/server.xml:/usr/local/tomcat/conf/server.xml
   - ./cluster/ROOT:/usr/local/tomcat/webapps/ROOT
lucee2:
  image: lucee/lucee4:latest
  volumes:
   - ./cluster/server.xml:/usr/local/tomcat/conf/server.xml
   - ./cluster/ROOT:/usr/local/tomcat/webapps/ROOT
lucee3:
  image: lucee/lucee4:latest
  volumes:
   - ./cluster/server.xml:/usr/local/tomcat/conf/server.xml
   - ./cluster/ROOT:/usr/local/tomcat/webapps/ROOT

```

***
##### Notes on volumes: 
* Docker machine auto mounts your users folder allowing you to do relative pathing in relation to your `docker-compose.yml` file. You can mount any folder using a full path as long as it's shared and properly mounted on the `docker-machine` VM. 
* On Windows paths would be something like `/c/Users/engage/sites/yourClusterProject` 
* I beleive on Mac it would be the same minus the `/c/..` portion. 
* You can manually share other folders with the Virtualbox instance and mount them within the VM, but I found that to be more trouble than it's worth. It's best to make sure you are running your project out of somewhere in your `/Users/` folder for now until they make this more robust via the `docker-machine` command.

### Reviewing the `docker-compose.yml` file
* The outer most level define the services. In our case we have a loadbalancer `lb` and 3 lucee services `luceeOne luceeTwo luceeThree`
* The `lb` is based on the offical nginx image and exposes 80
	* We mount our default nginx configuration for the localhost `default.conf`
	* Then we link the lb container with the 3 Lucee containers
* Each Lucee container is based off one of the official [Lucee Docker Images](https://hub.docker.com/u/lucee/) in this case the latest version 4 image.
* We sync our Server.xml file which defines our cluster membership Ramki once again for his Tomcat and NGINX tutorials.
*

### Feedback desired
This is a first go around trying to put Docker to work so please let me know if there are areas that can be done more efficiently and correctly. I assume I'm missing quite a few things in terms of "configuring" Lucee.

Thanks!
