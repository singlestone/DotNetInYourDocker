![](https://i.imgflip.com/15aaps.jpg)

Here is a great guide for getting your .NET Core app running in Docker without the Visual Studio tools:

https://github.com/docker/labs/blob/master/windows/dotnet-core/index.md

## Prep - To develop a Docker app with Visual Studio:

- Install docker toolbox for windows or mac osx (https://www.docker.com/products/docker-toolbox)
- Install the Visual studio Tools for Docker preview (https://visualstudiogallery.msdn.microsoft.com/0f5b2caa-ea00-41c8-b8a2-058c7da0b3e4)
- Right click on your project in visual studio solution explorer -> Add -> Docker Support
	- This will generate your powershell scripts that deploy your app into a Docker container and execute Docker commands in Docker to run your app.
	- **Note**: Powershell must be installed for Tools for Docker to be able to build and run your image.
- Run Docker Quickstart (this was installed as part of docker toolbox)
- Make sure your visual studio project root has been added to Shared Folders in the boot2docker.iso instance in VirtualBox
- In Visual Studio click the drop down next to the debug button and select "Docker"
- Run your application by pressing F5 or clicking the debug button (green arrow).
	- After your application builds, a powershell window should launch.
	- Powershell exits, debugging stops in Visual Studio, and your app is now running in Docker and should be accessible via the IP specified in your docker quickstart shell 
	(most likely 192.168.99.100 on Windows - use docker-machine ip MACHINE_VM on Mac to get this address)

You are now running your ASP.NET Core app in Docker. A docker image has been created for your app and can be monitored in the Docker shell or Kitematic (Docker GUI that comes with the Docker toolbox).

---

## Push your image to Dockerhub (From the docker cli):

- Type ```docker login``` from the command line and provide your user name and password as requested.
- Type ```docker images``` to find your image that was built with docker tools for visual studio.
- Type ```docker tag <ImageId> <Dockerhub Username>/<Dockerhub Repository>:<Your own tag name (ex: latest)>``` to Tag the image.
- Type ```docker push <newly tagged image name>```

Pull an image from Dockerhub and run locally:

- Type ```docker pull <your new docker image>```
- Run image
- If your container runs but is inaccessible from your browser, make sure it is exposing port 80. You can check this in Kitematic under the settings for your container or specify it in your docker run 
command.

---

## Integrating Redis with your .NET Core app

docker-compose.debug.yml (use spaces, not tabs or project won't build):

```yaml
-version: '2'

services:
  dockertest:
    build:
      context: ..
      dockerfile: Docker/Dockerfile.debug
      args:
      - CONTAINER_PORT=${DOCKERTEST_PORT}
      - SERVER_URLS=http://*:${DOCKERTEST_PORT}
    ports:
    - "${HOST_PORT}:${DOCKERTEST_PORT}"
    links:
    - redis
    volumes:
    - ..:/app
    environment:
      - REDIS_PORT_6379_TCP_ADDR=192.168.99.100
  redis:
    image: redis
    ports: 
        - "6379:6379"
```

- The environment: line adds an environment variable to .NET so it can communicate with Redis. Use the IP of your docker environment here.
- This will pull in the redis image once you build it and it exposes redis on port 6379
- Notice the links: line. This tells docker to allow the two images/containers to communicate.


## Useful links:

- Tutorials and VS update for docker tooling - https://www.docker.com/microsoft

- https://docs.docker.com/engine/installation/windows/

- http://www.hanselman.com/blog/BrainstormingDevelopmentWorkflowsWithDockerKitematicVirtualBoxAzureASPNETAndVisualStudio.aspx

- Visual studio ASP.NET 5 RC1: https://go.microsoft.com/fwlink/?LinkId=627627

- Run tests, edit and restart container, plus lots of other cool stuff - https://lostechies.com/gabrielschenker/2016/04/22/testing-and-debugging-a-containerized-asp-net-application/