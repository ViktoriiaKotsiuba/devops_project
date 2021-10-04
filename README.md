# DevOps Project by *Viktoriia Kotsiuba*

### *Preamble*
My goal in this project is to create a full CI/CD pipeline with *Git, Jenkins, Ansible, DockerHub, Docker to DEPLOY on a Docker container.* I did this project in stages, step by step complicating it. To begin with I created two AWS EC2 instances, one for Jenkins server and the second one for Web-server.

![alt-text](https://github.com/ViktoriiaKotsiuba/devops_project/blob/main/images/Project-4.png)

### Stage 1
In this stage I wanted to collaborate Git and Jenkins.

#### *Part 1*
Created Jenkins job on Jenkins console:
*  *Source Code Management:*
   * Repository: `https://github.com/ViktoriiaKotsiuba/hello-world.git`
   * Branches to build: `*/main`
* *Build:*
   * Root POM: `pom.xml`
   * Goals and options: `clean install package`

#### *Part 2*
Installed `deploy to container` plugin. This is need to deploy on tomcat server which I'm using. Also installed Maven plugin to configure, build and run Maven-based projects in Jenkins, because in java-project which I found on the Internet there was a file pom.xml.

To deploy build artifacts on tomcat server our Jenkins server need access. For this I setup credentials (which I was also setup on tomcat server):
`credentials` > `jenkins` > `Global credentials`> `add credentials`

Modified the same job I created in part 1 and added deployment steps.
* Post Steps
   * Deploy war/ear to container
      * war/ear files: `**/*.war`
      * containers: `Tomcat 8.x`

Run the job.

#### *Part 3*
Continuous Integration & Continuous Deployment (CI/CD). So I want to job trigger a build without any manual intervention and I modified job:
* Build Triggers
   * Poll SCM
      * schedule `H/2 * * * *`

This is it for *Stage 1*.

### Stage 2
In this stage I added Ansible server and created simple playbook to test.

#### *Part 1*
On Jenkins server installed "publish over SSH" plugin
* `Manage Jenkins` > `Manage Plugins` > `Available` > `Publish over SSH`

Enabled connection between Ansible and Jenkins
* `Manage Jenkins` > `Configure System` > `Publish over SSH` > `SSH Servers`
   * SSH Servers:
      * Hostname: `172.31.3.217`
      * username: `ansadmin` (I've created earlier on Ansible)
      * password: `********`

#### *Part 2*
Created a copyfile.yml on Ansible under `/opt/playbooks`
```sh 
# copyfile.yml
---
- hosts: docker
  become: yes
  tasks:
  - name: copy jar/war onto tomcat servers
    copy:
      src: /home/ansadmin/webapp/target/webapp.war
      dest: /home/ansadmin

```
### Stage 3
In this stage I completed full project with *Git, Jenkins, Ansible, DockerHub, Docker to DEPLOY on a docker container*

#### Part 1
I created Docker image on Ansible server through Jenkins job and pushed it onto DockerHub.

#### Part 2
I created *create_docker_container.yml* playbook. This get initiated by Jenkins job, run by Ansible and executed on Docker host.

```sh
# create_docker_container.yml
---
 - hosts: docker
   become: yes
   become_user: ansadmin

   tasks:
    - name: stop the container
      shell: docker stop webapp-image

    - name: remove container
      shell: docker rm -f webapp-image

    - name: remove image
      shell: docker image rm -f viktoriiakotsiuba/docker_project_3:latest

    - name: Creating a docker container
      shell: docker run -d --name webapp-image -p 8090:8080 viktoriiakotsiuba/docker_project_3:latest
```

#### Part 3
In this part I've tried to improvise to store docker images previous versions. I decided that the easiest solution would be maintaining version for each build. This can be achieved by using environment variables.
* `BUILD_ID` -  The current build id
* `JOB_NAME` - Name of the project of this build. This is the name you gave your job when you first set it up.
