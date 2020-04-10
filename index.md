
# INT332-DEVOPS Virtualization and Configuration Management

##Lab Assignment - 2


> By-
> Name: Kathi Raja Ravindra
> Redg No. 11702961
> Roll No. B37

> Submitted to-
> Dr. Vijay Kumar Garg







## Case Study 1:

> ### _Question_:

You work as a Devops Engineer in a leading Software Company. You have been asked to Dockerize the applications on the production server. The company uses custom software, therefore there is no pre-built container which can be used.
Assume the following things:

1. Assume the software to be installed is apache
2. Use an Ubuntu container

The company wants the following things:

1. Push a container to DockerHub with the above config.
2. The Developers will not be working with Docker, hence from their side you   will just get the code. Write a Dockerfile which could put the code in the custom image that you have built.
   
> _Approach/Scenario_:

I am supposed to deploy the company's portfolio website on apache server.
* Initially I will develop the required custom image on Developer Machine ('_DockerDevelop_') and push it into the repository.
*  Then I will deploy the code for website given by the company on Production Machine ('_DockerProduction_') using a _Dockerfile_ that uses the custom image.

> _Objectives_:

_On Developer Machine_:
* [ ] Custom image with apache installed.
* [ ] Push the image to repository.
  
_On Production Machine_:
* [ ] Dockerfile to copy code into the custom image.
* [ ] Deploy the website.



### On Developer Machine:

* Login into your _docker hub_ account using ```docker login```
* Pull the ```Ubuntu``` image and run a container using the following command.
 ![Image](Tutorial/Image1.png)

```bash
$ sudo docker pull ubuntu
$ sudo docker run -it ubuntu bash
```
![Image](Tutorial/Image2.png)

* Install apache server using following commands.
```bash
apt update
apt install apache2 -y
apt install apache2-utils -y
apt clean
```
> _utils_ is required for smooth functioning of the server.

![Image](Tutorial/Image3.png)

* Now commit the container and push the resulting image into Docker Hub repository using the following commands.
  
  ![Image](Tutorial/Image4.png)

```bash
$ sudo docker commit <containerId> <username/imageName>
$ sudo docker push <username/imageName>
```

### On Production Machine:

* Here on the production machine, we have Project code given as _ProjectPortfolio_ which contains the code for the website.
* Create a _Dockerfile_ to copy the code to our _Custom Image_ that we have pushed earlier
  
  ![Image](Tutorial/Image5.1.png)
> The above file uses the image that we pushed earlier viz. _krravindra/portfolio_ here.

* Now build by using this _Dockerfile_ 

  ![Image 1](Tutorial/Image5.0.png)

 > ```---tag``` help us in naming and assigning tag to the image that will be built by ```docker build``` viz. _portfolio_ here.

 > _Dockerfile_ builds in multiple layers as shown in above image (7 layers in above image)

 * Once the build is complete, we can see a new image named _portfolio_ is listed in ``` docker images``` 

![Image](Tutorial/Image6.png) 

* Now run the image and launch the _portfolio_ website on the server using the following command.
```bash
  $ sudo docker run -d -p 8080:80 <imageName>
```
> Open <u>localhost:8080/index.html</u> to view the website.

![Image](Tutorial/Image7.png)
 
 > ```-p 8080:80``` port maps the the _80_ port exposed by the custom apache server to _8080_ port on host machine.

 > ```--name``` tag will allow us to provide name for our container/instance of webserver image viz. _CompanyPortfolio_ here

``` -d ``` runs the container in detached mode.

* The Company Website is live and running as requested at [localhost](0.0.0.0:8080/index.html).
  
---  
>_Objectives Fullfiled_:

_On Developer Machine_:
* [x] Custom image with apache installed.
* [x] Push the image to repository.
  
_On Production Machine_:
* [x] Dockerfile to copy code into the custom image.
* [x] Deploy the website.

___

## Case Study 2:



You have been hired as a Devops Engineer in Grape Vine Pvt. Ltd. You have been asked to improve the way the company is managing their Docker containers. Following tasks have been assigned:

> ### _Question 1_:

 Deploy a sample HTML website in any apache container, and demonstrate how we can dynamically change content in the container by making changes on the host machine (Bind Mounts).

 > _Approach/Scenario_:

The company deploys their website [GrapeVine](google.com). The company later finds that the _About_ button on the webpage  is broken and is needed to be replaced by their portfolio webpage. The code for the website is deployed on the Apache server which is mounted on to the host system (_GrapeVine_). 

 > _Objectives_:
* [ ] Run apache webserver by mounting it to ther host machine
* [ ] Process the required modifications.
* [ ] Verify the changes.
  
  ---

> Demonstration of _bind mounts_ in docker.

* Run the apache image mounting it to the _code folder_ (_ProjectRubric_) on host machine (_GrapeVine_) using following commands.
  ```bash
  $ sudo docker run -d -p 81:80 --name <nameOfContainer> -v <hostPathToCode>:<pathInContainer> <imageName>
![Image](Tutorial1/Second1.png)

>  ```---name``` help us in naming and assigning tag to the image that will be built by ```docker run``` viz. _GrapeVine_ here.


> ```-p 81:80``` port maps the the _80_ port exposed on the apache server to _81_ port on host machine.

> ``` -d ``` runs the container in detached mode.

* Website is deployed on the server at [localhost:81/index.html](). However we can see that the _About_ button redirecting us to a broken page.  

![Image](Tutorial1/Second2.png)

> Demonstration of dynamic allocation (benefit of bind mount)

* To fix these we need to do modifications on a couple of files (_index.html_ & _portfolio/index.html_), before performing this let us verify that the files on both host and mounted directory in container are same using following commands.
    * Copying data present on container machine into _indexOld.html_ and _portfolio/indexOld.html_ on host machine.

    ```bash
    $ sudo docker exec GrapeVine cat index.html > indexOld.html
    $ sudo docker exec GrapeVine cat /var/www/html/portfolio/index.html > ./portfolio/indexOld.html
    ```
    > It is to be noted that the working directory of the container _GrapeVine_ is set to _/var/www/html/_.

    * Let us compare the derived files and host machine file to verify if they are same.
  
    ```bash
    $ diff index.html indexOld.html
    $ diff ./portfolio/index.html ./portfolio/indexOld.html
    ```
    > _NoOutput_ will be printed on console which shows the complete match of both the files as _diff_ command only prints the differences.

    * Successfully Verified. Now perform the required modifications.

![Image](Tutorial1/Second3.png)

* After performing the modifications, let us again compare both the files with previous derived files using the following commands.
  
```bash
    $ diff index.html indexOld.html
    $ diff ./portfolio/index.html ./portfolio/indexOld.html
```
> The output as shown in image lists the modifications in different files ( lists files using sha code ) seperated by '---' and enclosed in '< >'

> The line above '---' represents the modified code and the line below '---' represents code from derived files (old code). 

![Image](Tutorial1/Second4.png)

* To check if the modifications are also processed in the container, use the following code which can also be referred in the above image.

 ```bash
    $ sudo docker exec GrapeVine cat index.html > indexOld.html
    $ sudo docker exec GrapeVine cat /var/www/html/portfolio/index.html > ./portfolio/indexOld.html
    $ diff index.html indexOld.html
    $ diff ./portfolio/index.html ./portfolio/indexOld.html
```
> Once again the code from the containers is derived and the resultant is compared with the modified code.

> This time the diff command returns no output which shows that, the changes are also processed in the container since both files match completely.

* To view the modifications made, open [localhost:81/index.html]() in browser.

> Visual Modifications: Background Image changed, Fixed About button redirect.

![Image](Tutorial1/Second5.png)

 > _Objectives Fullfiled_:
* [x] Run apache webserver by mounting it to ther host machine
* [x] Process the required modifications.
* [x] Verify the changes.

> Demonstration of _bind mounts_ in docker successful.

---

> ### _Question 2_:
Deploy apache and nginx containers using Docker Compose, Apache should be exposed on Port 91 and nginx on port 92.

> _Approach/Scenario_:

An yaml file named ```docker-compose.yml``` is to be made which contains configuration about launching apache and nginx servers on required ports. To make apache image I will use a _Dockerfile_ and I will use _nginx_ image from the docker global repository for nginx server. Docker Compose version 3 will be used in the above said yaml file.



> _Objectives_:

* [ ] ```docker-compose.yml``` file to launch apache and nginx servers.

* [ ] Port 91 and Port 92 for _apache_ and _nginx_ servers respectively.

---

* On the host machine (_GrapeVine_), a _Dockerfile_ to build an _apache_ image is written as follows.

![Image](Tutorial2/third1.png)

* Create a file named, ```docker-compose.yml``` and write the following as shown in the image.

> _apache2_,_nginx_ listed under _services_ are the names of services that we want our docker compose file to perform.

> _ports_ helps to map the container/webserver to host. Port 91 of host is mapped to apache2 services's port 80 and port 92 of host is mapped to nginx service's exposed port 80.

> ```build``` under apache2 services specifies ```./apache``` which is a path to _Dockerfile_ while ```image``` under nginx service specifies ```nginx:latest``` which will be available on host/ global registry.

![Image](Tutorial2/third2.png)

```bash 
$cat docker-compose.yml

version: '3'
services:
    apache2:
            build: ./apache
            ports:
                - 91:80
    nginx:
            image: nginx:latest
            ports:
                - 92:80
```
* Execute the ```docker-compose.yml``` using following command:

```bash
$ sudo docker-compose up
```
![Image](Tutorial2/third3.png)

> The file populates the necessary services in layers (9 layers in image).

> A confirmation message ```done``` is printed on console for each of the service as shown in image.
![Image](Tutorial2/third4.png)

* To verify nginx server, open [localhost:92]() in browser.
![Image](Tutorial2/third5.png)

* To verify apache server, open [localhost:91]() in browser.

_Important Note: Company's page is deployed on the apache server at port 91_
![Image](Tutorial2/third6.png)

* Verify the containers and the activity on console.
![Image](Tutorial2/third8.png)

> Whenever the page is hit, the log prints the ```GET``` request which can be seen in the first half of image.

> Two containers (nginx and apache) with predefined _commands_ are running as listed in the ```docker ps``` command in image.

> _Objectives Fullfiled_:

* [x] ```docker-compose.yml``` file to launch apache and nginx servers.

* [x] Port 91 and Port 92 for _apache_ and _nginx_ servers respectively.

---
> ### _Question 3_:
 Initialize a Docker Swarm Cluster, and deploy two ubuntu containers in a overlay network. Demonstrate they can communicate with each other, by pinging them.

> _Approach/Scenario_:

I will create a _docker network_ and initiate a _docker swarm_ cluster. Then after I will run a _docker service_ to populate the _swarm_ with 2 ubuntu containers and try pinging them.



> _Objectives_:

* [ ] Create an overlay network.

* [ ] Instantiate docker swarm and add nodes.

* [ ] Use docker service to populate the swarm.

* [ ] Show connection over overlay network by means of ping.
---
* Two _EC2_ instances _Master Node_ and _Worker Node_ are spun on _AWS_ platform.

![Image](Tutorial3/fourth1.png)

* Docker is installed and _docker swarm_ is instantiated on _Master Node_ which will be the _Leader_ of the swarm.

```bash
$ sudo docker swarm init
```
![Image](Tutorial3/fourth2.png)

* On Worker node, the token collected from Leader is used to join the Worker into swarm.

```bash
$ sudo docker join --token <tokenID>
```

![Image](Tutorial3/fourth3.png)

* List the docker networks in Leader node using the following command.

```bash
$ sudo docker network ls
```
![Image](Tutorial3/fourth4.png)

* To create and inspect an Overlay network, use the following commands.

```bash
$ sudo docker network create --driver overlay MyNetwork 
```
![Image](Tutorial3/fourth5.png)

> _MyNetwork_ is the name of the overlay network.

```bash
$ sudo docker network inspect MyNetwork
```
![Image](Tutorial3/fourth6.png)

> Note that this network domain is on the _Subnet: 10.0.2.0/24_, every container that uses this network will be assigned an address in this subnet.

* To start a service, use the following command as shown in the image.

```bash
$ sudo docker service create --name Pingservice --replicas 2 --network MyNetwork ubuntu sleep 2d
```
![Image](Tutorial3/fourth7.png)

> _Pingservice_ is the name of the service created.

> Two replicas of ubuntu image are spun according to this service over the overlay network _MyNetwork_.

> Two ubuntu containers are running in the nodes of swarm, with startup command ```sleep 2d```.

* To view the networks these are configured in, use the following command.

```bash
sudo docker network inspect MyNetwork
```
![Image](Tutorial3/fourth8.png)

> _Leader Node_ is on ip 10.0.2.250/24 belonging to MyNetwork subnet.

>_Worker Node_ is on ip 10.0.2.251/24 belonging to MyNetwork subnet.

* To ping _Master Node_ execute the follwoing on _Worker Node_

```bash 
$ sudo docker exec -it <containerID> bash
ping 10.0.2.250
```
![Image](Tutorial3/fourth9.png)

* To ping each other simultaneously,

> On MasterNode:

```bash
$ sudo docker exec -it <containerID> bash
ping 10.0.2.251
```
> On WorkerNode:

```bash
$ sudo docker exec -it <containerID> bash
ping 10.0.2.250
```
![Image](Tutorial3/fourth10.png)

* Successfully executed ping on both nodes of the swarm.

> _Objectives Fulfiled_:

* [x] Create an overlay network.

* [x] Instantiate docker swarm and add nodes.

* [x] Use docker service to populate the swarm.

* [x] Show connection over overlay network by means of ping.

---
> ### _Question 4_:
Install Jenkins and create any sample job to demonstrate its operation.

> _Approach/Scenario_:

First part, shows the installation of jenkins on Ubuntu 18.04. Second part builds a basic job on jenkins to support Continous Integration of a sample java code.


> _Objectives_:

* [ ] Install Jenkins on ubuntu 18.04.
* [ ] Explain Jenkins by creating a sample job.
---
* Use following commands to install jenkins on Ubuntu.

```bash
$ sudo apt update
$ sudo apt install openjdk-8-jdk
$ wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
$ sudo apt update
$ sudo apt install jenkins
$ systemctl status jenkins
$ sudo ufw allow 8080
```
Open [localhost:8080]() after executing above.

* Find the password in host using 
```bash 
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
![Image](Tutorial4/Initial1.jpg)

* Proceed to add required plugins and create login credentials.
![Image](Tutorial4/Initial2.jpg)

* You can edit the Jenkins URL or leave the auto generated and then you are ready.
![Image](Tutorial4/Initial3.jpg)

![Image](Tutorial4/Initial4.jpg)

> Demonstration of Jenkins

* Create a sample helloWorld.java program as shown in image.

![Image](Tutorial4/Main1.png)

* Select _Create Job_/_New Item_ on jenkins dashboard and name it.

![Image](Tutorial4/Main2.png)

> _JavaTest_ is the name of our job which is listed as _Freestyle project_.

* Give a decent description to the job and select the option _Build Periodically_ to automate the building process.
![Image](Tutorial4/Main3.png)

> ```* * * * *``` says that a new build will be made every other minute.

![Image](Tutorial4/Main4.png)

* Select _ExecuteShell_ in the Build section and enter the following code as shown in image to test the HelloWorld.java program we created in the hostmachine.
![Image](Tutorial4/Main5.png)
![Image](Tutorial4/Main6.png)
> The Execute Shell lists the commands to be executed on every build.

* We created our first job _JavaTest_ .
![Image](Tutorial4/Main7.png)
> On left bottom corner you can see the history of our builds.

![Image](Tutorial4/primary.png)
> The dashboard view of the job is shown in above image.

> Blue circle means the build passes the test, and the Sun beside it represents the health of the project (current 100%).

* To view output of a build, 
Click ItemName (JavaTest) --> Build (right click) --> select _ConsoleOutput_.
![Image](Tutorial4/Main8.png)
![Image](Tutorial4/Main9.png)

* To try failing our job, edit the HelloWorld.java as follows which will result in an error, and then let us check the build history and failed job output

![Image](Tutorial4/Main10.png)

>Removed " in println function which will throw an error. 

> Verify this on Jenkins...

![Image](Tutorial4/Main11.png)
> A pink circle indicates that our job failed

![Image](Tutorial4/Main12.png)
> The output log shows the error and _FAILED_ build at the end of output as shown in the image.

* Verify the failed build in dashboard.

![Image](Tutorial4/Main13.png)

>A pink ball notifies the test failed.

> The thunderstorm cloud represents poor health of our project. (only 20% builds successful).

* Modify the HelloWorld.java and Check the Jenkins dashboard again.

![Image](Tutorial4/Main14.png)

> After modification, the ball is blue which shows that our testcases in the latest build have passed.

>The Thunderstorm cloud started to convert into a Normal cloud which signifies improved health of our toal project (80% builds pass test).

* Now, click the _BuildHistory_ option on left side in the dashboard to view the whole project against a graph.

![Image](Tutorial4/Main15.png)

> The image shows a couple of jobs in the build that is released after build 6 failed till build 11 after which it is corrected.

> This shows an overall view of how our code/project did over a period of time.

![Image](Tutorial4/Main16.png)

> A healthier dashboard is visible again after 16 builds, the last failed build is 11 and our health is at 100% (Sun).

* In this way one can continously build code and commit changes without the worry of testing it. Jenkins automates the process of Continously integrating the new code without developer intervention and notifies if there is a failure in the recent builds so that one could verify the problem and rectify it. Projected on a large view this tool supports the Continous Integration pipeline where large chunks of codes can be updated simultaneously to the production server leaving testing to automation which in turn saves massive amount of hours spent on testing code.

> _Objectives Fullfiled_:

* [x] Install Jenkins on ubuntu 18.04.
* [x] Explain Jenkins by creating a sample job.
---

Can be viewed at [GitHub](https://github.com/KR-Ravindra/DockerProject.git)
