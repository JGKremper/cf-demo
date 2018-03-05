# Cloud Demo

This application is a demo cloud native application using Spring Boot and Cloud Foundry.
We cover the use of [Jenkins](https://jenkins.io/) and [Concourse](https://concourse.ci/).
<!-- TODO: Travis CI? -->

## Jenkins Setup

Download and run Jenkins via Docker:

    docker run -p 8080:8888 -p 50000:50000 jenkins/jenkins:lts

For convenience, you can also use the bundled `docker-compose.yml` setup:

    docker-compose up

On startup, an autogenerated admin setup password is printed the console.
Copy this, then navigate to http://localhost:8888/ or http://192.168.99.100:8888/ depending if you're using Docker natively or via something like Docker Toolbox respectively.
Enter this initial password and procede to the next page.
Click the "Select plugins to install" option, then on the next page, check the box for "JUnit", then click "Install" at the bottom.
When this completes, you'll be prompted to create your first user.

From the home page, navigate to "Manage Jenkins" on the sidebar.
Click on "Global Tool Configuration", then scroll down to the Maven section.
Click "Add Maven", then give it a name "apache-maven-3.5.2" and "Install from Apache" using that version.
Then click "Save" and then go back to "Manage Jenkins" on the sidebar.
Now go to "Manage Plugins" and navigate to the "Available" tab.
Search for "cloud foundry" in the filter box, select the Cloud Foundry plugin, then click "Install without restart".

Next, click "New Item" on the sidebar.
Enter a project name, generally matching the naming scheme of your repository or application name.
This is used in URLs and such, so make it convenient for that.
We can set a nicer display name later.
Select "Multibranch Pipeline" or "Pipeline" and then "OK".

On the next page, add a branch source for your GitHub project.
Add credentials from here by adding both your GitHub username/password as an entry with the id "github" and your Cloud Foundry username/password as an entry with the id "pcf".
Choose your GitHub credentials, then enter the username/organization and repository name corresponding to your project.
Click save, and Jenkins will check out your project.
Using the provided Jenkinsfile, this should build and deploy automatically.

## Concourse Setup

First, generate SSH keys for Concourse web and slaves via the included script:

    ./init-concourse-keys.sh

Download and run Concourse via Docker:

    docker-compose up

Using the `fly` CLI tool from Concourse, log in to your instance (replace `localhost` with `192.168.99.100` if using Docker Toolbox):

    fly -t lite login -c http://localhost:8080 -u concourse -p changeme

Next, edit `pipeline.yml` and change the username and password attributes to your Cloud Foundry details.
Then, upload that configuration to Concourse via:

    fly -t lite set-pipeline -c pipeline.yml -p cf-demo

Finally, unpause the pipeline to start it:

    fly -t lite unpause-pipeline -p cf-demo

Now you can navigate to http://localhost:8080/teams/main/pipelines/cf-demo/ to see the status of this pipeline.
(Docker Toolbox users navigate to http://192.168.99.100:8080/teams/main/pipelines/cf-demo/ instead).
