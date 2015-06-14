# Ice, Ice Baby 

This project creates a container that runs [Netflix's AWS Usage tool, Ice](https://github.com/Netflix/ice) and supplies a Nginx proxy container as well. From [Netflix's Ice repository](https://github.com/Netflix/ice) introduction:

> Ice provides a birds-eye view of our large and complex cloud landscape
> from a usage and cost perspective. Cloud resources are dynamically
> provisioned by dozens of service teams within the organization and any
> static snapshot of resource allocation has limited value. The ability
> to trend usage patterns on a global scale, yet decompose them down to
> a region, availability zone, or service team provides incredible
> flexibility. Ice allows us to quantify our AWS footprint and to make
> educated decisions regarding reservation purchases and reallocation of
> resources.
> 
> Ice is a Grails project. It consists of three parts: processor, reader
> and UI. Processor processes the Amazon detailed billing file into data
> readable by reader. Reader reads data generated by processor and
> renders them to UI. UI queries reader and renders interactive graphs
> and tables in the browser.

More information and screenshots can be found on the [project's git page](https://github.com/Netflix/ice). 

## What is this repository?

This repository seeks to ease the installation and configuration of Ice. In addition to the application container, this repository configures a nginx proxy which also helps fix URI issues I had when accessing Ice directly. After following these directions you should be able to connect to your server's IP address or FQDN over port 80 and access the Ice application.

# Getting Started

## Prerequisites 

 - Sign up for Amazon's programmatic billing access [here](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/detailed-billing-reports.html) to receive detailed billing(hourly) reports. Verify you receive monthly billing file in the following format: <accountid>-aws-billing-detailed-line-items-<year>-<month>.csv.zip.
 - [Docker](https://docs.docker.com/installation/) and [Docker Compose](https://docs.docker.com/compose/install/) installed.

## Docker Setup

 - Create the docker-compose file: `cp docker-compose-template.yml docker-compose.yml` 
 - Open docker-compose.yml and add the AWS Access Key ID and Secret Key that has access to the s3 billing bucket: `vi docker-compose.yml`

	    ice:
	      build: ice
	      command: |
	        -Djava.net.preferIPv4Stack=true
	        -Djava.net.preferIPv4Addresses
	        -Dice.s3AccessKeyId=<s3AccessKeyId>
	        -Dice.s3SecretKey=<s3SecretKeyId>
       
 - Create the configuration file that will be mounted to the container: `cp ice/assets/sample.properties ice/assets/ice.properties`
 - Open ice.properties and configure a basic setup by updating the following: `vi ice/assets/ice.properties` 
    
	    # s3 bucket name where the billing files are
	    ice.billing_s3bucketname=
	    
	    # Your company name
	    ice.companyName=
	    
	    # s3 bucket name where Ice can store output files
	    ice.work_s3bucketname=
	    
	    # Your AWS account number. You can also replace "production" with your own identifier 
	    ice.account.production=

More information on the configurations can be found on the [project's git page](https://github.com/Netflix/ice). 

## Docker Compose

 - When you have completed the previous steps, issue `docker-compose up` This will start the containers in the forground so you can see if there are any errors.
 - Once everything looks good and you can access the UI issue `docker-compose up -d` to run the containers in the background.

## Base Docker Containers

- The nginx container is pulled from the [official nginx Docker Hub repository](https://registry.hub.docker.com/_/nginx/).
- The Ice container's base image is a [Java 7 container](https://registry.hub.docker.com/u/jonbrouse/java/) which is part of an automated build repository that I maintain.

# Upstart Job

I've included an Upstart job in the `init` directory of this repository. This will allow you to start the containers with `start ice` and stop them by running `stop ice`.  This will also start your containers at boot.

1. Copy `init/ice.conf` to your host's `/etc/init/` directory
2. Edit the the job `vi /etc/init/ice.conf` and change the path to the docker-compose file
	
	    pre-start exec /usr/local/bin/docker-compose -f /path/to/your/docker-compose.yml up -d

		post-stop exec /usr/local/bin/docker-compose -f /path/to/your/docker-compose.yml stop

4. Reload the job controller `initctl reload-configuration`
