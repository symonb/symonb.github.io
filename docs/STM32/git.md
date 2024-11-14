---
layout: default
title: Git for beginners
permalink: /docs/STM32/git
nav_order: 1
parent: STM32
toc: true
---

<!-- comment or image allows {: .no_toc} to work correctly  (don't ask me why) -->

[![image]()]

{: .no_toc }

# Git, GitHub or GitLab what is a difference?

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

# Introduction
Git by definition is a distributed version control system  - in other words it is a software that tracks versions of the files in your project. When you make an update old version of the code or other files are kept and you can always go back to them, check what has changed between different saved versions. Thanks that you can really smoothly develop and add new feature to your code within one project also git keeps only the changes so you don't waste space for coping the same code for each version.
This can be done locally on your computer (you don't need anything else than git software) or on the online server (remote repository) of other softwares which are using *git* as a backbone of their structure and you use the same commands but besides that you have other feature which helps with collaboration on the projects and keeping everything on remote server so you have a backup in case of damaging your local project. And for that are platforms as GitHub or GitLab. You can also add issues to the project write a description, documentation, set up the tasks for the team etc.

# git locally
At first, we will start with the basic git commands which should give you general overview how it works and how to start. First step is to install git on your machine - for that follow the steps on the official [website](https://git-scm.com/downloads). If you use the installer as it is, Git will install in the default path (on Windows _C:\Program Files\Git_ and */usr/bin/* for Ubuntu). It is not a problem but if you want to specify a directory you need to open the cmd window in the folder where you’ve got git installer and type `git_installer_name.exe /DIR=“your\path\to\preferred\location”`. 
After that you should be able to check the version with the command ```git --version```. If git command is recognized thats good if not you probably need to add it to the environmental variables (to the ```PATH```). 

## Repository
Now lets initialize the new repository - the new project in which git should keep track of the changes that were made. ```git init 'name-of-the-repository'```
Then you can check the status of the repository - ```git status```, everything should be up to date and you should be in the "*main*" branch 
[![image](images/git/Screenshot%20from%202024-11-14%2016-31-16.png)](images/git/Screenshot%20from%202024-11-14%2016-31-16.png)<custom_caption>Initialize new repository and check status</custom_caption>
We have our project created now it is time to add first file. Inside a repository (directory you've just created) just create any file - it can be .txt, python, .c, image or PDF. I go with classic ```HelloWorld.py```. 
![image](images/git/)](images/git/)<custom_caption></custom_caption>
## Commits 
![image](images/git/)](images/git/)<custom_caption></custom_caption>
## Branches
![image](images/git/)](images/git/)<custom_caption></custom_caption>
# Remote repository
![image](images/git/)](images/git/)<custom_caption></custom_caption>
# Useful app

# Workflow 