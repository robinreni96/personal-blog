---
title: Deploying JupyterHub in Heroku
author:
  name: Robin Reni
  link: https://github.com/robinreni96
date: 2022-02-26 20:00:00 +0800
categories: [Cloud]
tags: [Python]
math: true
mermaid: true
image:
  src: /assets/img/posts/jupyter.png
  width: 800
  height: 500
---
For collaborative working and rapid experimentation's in Data Science, Jupyter Notebooks play a major role. Data Scientists/ ML Engineer or anybody who works with data science love jupyter notebook because of its simple UI, running environment and its integration support.

Most of them personally use their local setup to run jupyter labs or notebook for their experimentation. What if you need to collaboratively work with the set of developers and handle multi user support. Yes we prefer to deploy the Jupyter in cloud environments like AWS, GCP or Azure etc. But here is the catch, all these cloud providers are costly and for initial phases it prices to much to set it up also Jupyterlab or Notebook does not support multiuser.

In this blog we gonna deploy **JupyterHub** in **Heroku**(cloud provider) which cut down the cost and also easy to deploy.

> Note: There are many ways of doing this. I am explaining the best workflow which I experimented

### JupyterNotebook vs JupyterLab vs JupyterHub
**Jupyter Notebook** is a web-based interactive computational environment for creating Jupyter notebook documents.

**JupyterLab** is the next-generation user interface, including notebooks. It has a modular structure, where you can open several notebooks or files (e.g., HTML, Text, Markdowns, etc.) as tabs in the same window

**Jupyterhub** is for servers, and let you have jupyter notebook for an entire office or classroom or team.

### Prerequisites
#### 1. Github Account:
If you have a github account please create a repo as per your custom name if you dont have github account please create a [account](https://github.com/join) and create a repo.
#### 2. Heroku Account:
If you have a heroku account please create a dyno app as per your custom name if you dont have github account please create a [account](https://signup.heroku.com/) and create a dyno app.

### Workflow
![Clients invoke a function synchronously and wait for a response.](/assets/img/posts/jupyterhub_workflow.png)

### Connect Github Repo with Heroku App
* At this stage you will be having a github repo and a heroku app created. In the heroku app go to Deploy option dashboard -> select the deployment method -> select Github. You will see something like this.

  ![heroku_jupyterlab_github](/assets/img/posts/heroku_jupyterhub_git.png)
* Give the repo name is connect to Gihub field -> **Connect**
* In the Automatic deploys, select **Enable Automatic Deploys**. After the above two steps you will see something like this.
![heroku_gitlab_connect](/assets/img/posts/heroku_gitlab_connect.png)
Now when we push code to github it will be auto deployed to Heroku app (Mini CI/CD Pipeline)

### Prepare the docker and config files
Clone the github repo to your local and add the below files
* Create Jupyter Config file(**jupyterhub_config.py**). You can add additional settings if you are familiar with configuring jupyter.
  ```python
  from jupyterhub.spawner import SimpleLocalProcessSpawner

  # setting a dummy user admin for now
  c.JupyterHub.authenticator_class = "dummy"
  c.DummyAuthenticator.password = "admin"

  # using simplelocalspawner for now
  c.JupyterHub.spawner_class = SimpleLocalProcessSpawner
  c.Spawner.cmd = ['jupyter-labhub']

  # for creating new users
  c.LocalAuthenticator.add_user_cmd = ['python3','/app/analysis/create-user.py','USERNAME']
  c.LocalAuthenticator.create_system_users = True
  ```
* Create the create user helper script(**create-user.py**)
  ```python
  import crypt
  import os
  import sys

  if __name__ == '__main__':
      if len(sys.argv) <= 1:
          sys.stderr.write('Usage : create-user.py <username>\n')
          sys.exit(1)
      if 'DEFAULT_USER_PASSWORD' in os.environ:
          default_password = os.environ['DEFAULT_USERS_PASSWORD']
      else:
          default_password = 'remember.change.it'

      username = sys.argv[1]
      os.system("useradd -p "+crypt.crypt(default_password,"22")+" -m "+username)
  ```
* Create the Dockerfile
  ```Dockerfile
  FROM ubuntu:latest

  # for heroku the ports association is dynamic
  ARG port
  ENV PORT=$port

  # if needed you can rename the workdir
  WORKDIR /app/analysis

  # Install python, node, npm packages
  RUN apt-get upgrade -y && apt-get update -y && apt-get install -y python3-pip && pip3 install --upgrade pip
  RUN apt-get -y install curl gnupg
  RUN apt-get -y install git
  RUN curl -sL https://deb.nodesource.com/setup_14.x  | bash -
  RUN apt-get -y install nodejs
  RUN npm install
  RUN npm install -g configurable-http-proxy

  # Install python packages add jupyter plugin packages if you are using I have added dask and git for my experiment
  RUN pip3 install jupyterhub && \
      pip3 install --upgrade notebook && \
      pip3 install oauthenticator && \
      pip3 install pandas scipy matplotlib && \
      pip3 install "dask[distributed,dataframe]" && \
      pip3 install dask_labextension && \
      pip3 install --upgrade jupyterlab jupyterlab-git && \
      jupyter lab build

  # add user admin to create login for your jupyterhub
  RUN useradd admin && echo admin:change.it! | chpasswd && mkdir /home/admin && chown admin:admin /home/admin

  # adding python supporting scripts
  ADD jupyterhub_config.py /app/analysis/jupyterhub_config.py
  ADD create-user.py /app/analysis/create-user.py

  # expose the port
  EXPOSE $PORT

  # run the jupyter hub feel free to add your arguments needed
  CMD jupyterhub --ip 0.0.0.0 --port $PORT --no-ssl
  ```
* Create heroku yaml which says to heroku how to execute these files (**heroku.yaml**)
  ```yaml
  build:
    docker:
      web: Dockerfile
  ```
