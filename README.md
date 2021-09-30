# DevOps Project by *Viktoriia Kotsiuba*

### *Preamble*
My goal in this project is to create a full CI/CD pipeline with *Git, Jenkins, Ansible, DockerHub, Docker to DEPLOY on a Docker container.* I did this project in stages, step by step complicating it. To begin with I created two AWS EC2 instances, one for Jenkins server and the second one for Web-server.

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
