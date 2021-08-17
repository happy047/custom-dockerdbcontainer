# custom-dockerdbcontainer
This repository focuses on explanation for creating a custom db docker of a particular project in github and pulling the container using lando

## Table of contents

  * [Goal](#goal)
  * [Tools and Prerequisites](#tools-and-prerequisites)
  * [Github and GHCR Setup](#github-and-ghcr-setup)
  * [Local environment setup](#local-environment-setup)
  * [Custom DB Docker image building](#custom-db-docker-image-building)


### Goal

The idea is to upload our custom project oriented DB in Github container Registry portal and include that image volumn to our lando yml file. By implementing that, lando will automatically pull the latest db that is uploaded in the [Github Container Registry](https://ghcr.io) portal.

### Tools and Prerequisites

The following tools are required for setting up the setup. Ensure you are using the latest version or at least the minimum version mentioned below.

* [Composer](https://getcomposer.org/download/) - v2.0.11
* [Docker](https://docs.docker.com/install/)  - v20.10.5
* [Lando](https://docs.lando.dev/basics/installation.html) - v3.0.28
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) - v2.25.1

*Note: Ensure you have sufficient RAM (ideally 16 GB, minimum 8 GB)*

### Github and GHCR Setup

First things first, this documentation is for the users who have the project in github portal. The user needs to have logged in to github and [ghcr](https://ghcr.io) to upload the 
DB docker image. The user needs to authenticate first using GITHUB_TOKEN for best security practices. In order to do that, there is a great article written in Github Docs.
Please go through the steps of following article for authenticating container registry :
* [Working with the Container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

For registering your personal token in github, go to the [settings menu](https://github.com/settings/profile) of your Github portal and then go to the
[Developer Settings Menu](https://github.com/settings/apps). There, navigate to Personal Access Token menu and create a new token. Make sure you provide sufficient access for that token.
For instance, you need to check the *repo* checkbox for accessing private repositories and commits and also check the *write:packages* checkbox to upload containers.
Additionally, you can check the *user* checkbox to manage the user level permissions. Thats it, you have your token set.

*Note: You must write the alphanumeric token to somewhere safe after creation as it wont show to anywhere else. I would suggest to forward it to self via email.*

### Local environment setup

Considering you have cloned your project repository in your local, just authenticate in the [GHCR](https://ghcr.io) portal using the steps mentioned in :
* [Working with the Container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)

### Custom DB Docker image building

Considering you have worked on your local project and updated your database, lets move to build up your image of your db. Its time to create a folder named DB in your project root folder
and a file named Dockerfile. Make sure, you include these in your gitignore file so that they are not pushed to remote repository.

Now export your modified db as sql. By default, lando exports db as dbname.gz or dbname.sql.gz

```bash
$ lando db-export customDB.sql
```
This will export the db as customDB.sql.gz
You need to unzip the sql file using the following command

```bash
$ gunzip customDB.sql.gz
```

After that, you need to move the unzipped sql file to DB folder and then write the following in the Dockerfile created at the root folder of your project

```bash
# The base db image, it can be mysql also, check which database image is running using docker ps command
FROM bitnami/mariadb
# Add the database of your project
ENV MYSQL_DATABASE customDB
# Copy the DB to docker-entrypoint-initdb.d/ which will be automatically executed during container startup
COPY ./DB/ /docker-entrypoint-initdb.d/
```
You are now set to build your customdb image for your project. Run the following command for building the db image with tag latest

```bash
$ docker build -t ghcr.io/github_username/customDB:latest .
```
In the above command, replace github_username with your username and customDB with your project db. Docker will automatically build the image and will show a success notification after
building the db image. To verify that the image has been created, run docker images command to check the docker images list. Now, your docker db image has been created and you need to push 
the image in [GHCR](https://ghcr.io) portal. For that, you need to run the following command:
```bash
$ docker push ghcr.io/github_username/customDB:latest
```
After running the command, you will be prompted to authenticate with your github username and password [password will the personal token which you have created earlier].
Now you have uploaded the image in [GHCR](https://ghcr.io) portal. Its time to pull the image through lando file so that it automatically pull your latest uploaded project db from github.
Just replace your database service code with the following code in your lando file:

```bash
  database:
    type: mariadb
    overrides:
      image: ghcr.io/happy047/dlatest
```
Now, each time any user with sufficient permission clones the repository and run lando start command, it will automatically pull the custom db image you uploaded.
