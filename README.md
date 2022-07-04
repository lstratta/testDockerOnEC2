---
title: Docker on EC2
description: Install docker on an EC2 instance. Then build and run an image created from a Dockerfile inside a GitHub repository before pushing the image to the Docker Hub container registry.
---

# Build and run a Docker container image on an EC2 instance

### Prerequisites
- [x] Docker account
- [x] Access to AWS EC2
- [x] GitHub account
- [x] PuTTY

### Setup EC2 instance
1. Create a key pair
2. Create a security group with inbound rules set to allow SSH from your IP address and outbound rules to allow HTTP(S) to the internet (0.0.0.0/0).[^1]
3. Launch an EC2 instance with the key pair and security group you created.
4. Use PuTTY to connect to your EC2 instance. [^2]
   Two things to do: 
    1. Set the host name of your EC2 instance
        - In the 'Category' pane, click 'Session' and set Host Name to "ec2-user" + "@" + Public IPv4 DNS of EC2 instance. 
        - For example: ec2-user@ec2-54-91-206-43.compute-1.amazonaws.com
        (The Public DNS should appear once your EC2 instance is in the "running" state.)
    1. Open the private key paired with the EC2 instance's public key. 
       - Open 'Connection', then 'SSH' and click 'Auth':
         Open the private key you downloaded when you created the key pair.

From this point onwards, we will be working inside the PuTTY client.

### 1. Setup

Update packages on EC2 instance

```bash
sudo yum update -y
```

Install Docker

```bash
sudo yum install docker -y
```

Install Git
```bash
sudo yum install git -y
```

### 2. Build a container image from a Dockerfile

Docker needs to be running before we can build an image from a Dockerfile.

Check if docker is running

```bash
sudo service docker status
```

Start docker if docker is not running

```bash
sudo service docker start
```

Build a container image from a Git repository. Either use [this one](https://github.com/tangjm/testDockerOnEC2#main) I made  or create your own.

```bash
sudo docker build -t my-app:v1 https://github.com/tangjm/testDockerOnEC2#main
```

The argument to `docker build` is the build context. This is a set of files used for generating your image. They should reside within the same directory as your Dockerfile. In this case, we're passing a URL to use a remote directory for our build context. The suffix `#main` indicates that we want the main branch version of our repository. See [docs](https://docs.docker.com/engine/reference/commandline/build/#git-repositories) for more on this.

The build context is first sent to the Docker daemon. Then the instructions within your Dockerfile are executed in order. Finally, the container image is built.

Check that the image was built successfully

```bash
sudo docker images
```

Try out the `docker tag` command by creating another tag

```bash
sudo docker tag my-app:v1 second-app:v1
```

Run the image in a container. This should print "Hello world!" to the terminal.

```bash
sudo docker run my-app:v1
```

### 3. Pushing images to container registries

To push to `docker.io`, you will need a docker account, register for one [here](https://hub.docker.com/)

Before pushing your image, make sure to login first
```bash
sudo docker login
```

Then tag the image you want to push

```bash
sudo docker tag my-app:v1 <your_docker_username>/my-app:v1
```

For example, for me, it would be this:

```bash
sudo docker tag my-app:v1 tangjm5/my-app:v1
```

Finally, push the tagged image:

```bash
sudo docker push <your_username>/my-app:v1
```

Verify the push by going back to [docker hub](https://hub.docker.com/).

A new repo should have appeared!



[^1]: We allow SSH from your IP address so that you can connect to your EC2 instance via SSH using PuTTY. The outbound rules are there so that you can access the internet to install Docker and Git.

[^2]: [Detailed instructions here](https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/#using-putty-to-connect-to-your-instance)
