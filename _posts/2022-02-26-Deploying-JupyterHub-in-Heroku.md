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

### JupyterNotebook vs JupyterLab vs JupyterHub
**Jupyter Notebook** is a web-based interactive computational environment for creating Jupyter notebook documents.

**JupyterLab** is the next-generation user interface, including notebooks. It has a modular structure, where you can open several notebooks or files (e.g., HTML, Text, Markdowns, etc.) as tabs in the same window

**Jupyterhub** is for servers, and let you have jupyter notebook for an entire office or classroom or team.
