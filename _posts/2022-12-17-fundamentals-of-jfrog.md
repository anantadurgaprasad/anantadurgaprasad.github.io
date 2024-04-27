---
title: "Fundamentals of Jfrog"
layout: post
categories: jfrog
tags: jfrog  
---

This article is part of Jfrog series:

1.  Fundamentals of Jfrog
2.  [Demo maven project with Jfrog + jenkins pipeline integration.](https://medium.com/@anantadurgaprasadar/maven-project-with-jfrog-artifactory-jenkins-pipeline-integration-71ee77d812e3)
3.  Configuring other project to use jfrog Artifact.

{% include toc %}


What is Jfrog Artifactory?
--------------------------

*   Jfrog Artifactory is a repository manager created by JFrog . As the name suggests is used to **manage artifacts and it’s dependencies**.
*   It is used to store artifacts or binaries produced from build.
*   **Github is used to store and manage source code. Similarly Artifactory is used to store and manage build artifacts or binaries.**
*   There are other alternative to Jfrog Artifactory like [Nexus Repository Manager](https://www.sonatype.com/products/nexus-repository)

2\. JFrog Trial account Creation :
----------------------------------

There are two ways to work with Jfrog Trial account :

1.  [Using Self Hosted open source software](https://jfrog.com/community/download-artifactory-oss/)
2.  [Using JFrog Artifactory SaaS service.](https://jfrog.com/start-free/#saas)

I choose the later one simply because it’s simple and easy to set up.

When you login the following page appears

for now close it and explore the jfrog console. we will be using it in the entire jfrog series.

3\. Fundamentals of Jfrog :
---------------------------

**Jfrog has three types of repositories :**

click on the type of repo you want → select project type → give repo name and click create repository.

1.  **Local Repository :**

It is a repository where the artifacts or binaries is actually stored.

2\. **Remote Repository :**

It is a repository which is configured to resolve dependencies from a central repository or any other remote repository (could be another jfrog Artifactory server also) . It caches them for later use as well. **It acts as a proxy for resolving dependencies of the code being built.**

**while creating remote repository you can define the url from which dependencies should be resolved from.**

3\. **Virtual Repository :**

It is not actually a repository as the name suggest but an aggregate of local and remote repositories which is used to represent all those repos it includes through this single repository.

**you can have any number of local or remote or virtual repositories in your virtual repository**

Here you have to manually select the repoistories to be the virtual repo created.

Ex . If a project is configured to look in virtual repository to resolve dependencies . It will look in all the repos that are present under virtual repo to resolve them.

**Maven Project Repo creation :**

Click on quick setup → select the project type (maven) → give the repo prefix and click create .  
Here we see all the repositories required for our project being created.

**Local Repositories** — to store the build artifacts for dev and prod env.

**Remote Repositories** — to resolve dependencies

**Virtual Repositories** — to be used by other projects which has this project as dependency .

**Virtual Repositories has following repos :**

**trial-libs-snapshot** - trial-libs-snapshot-local + trial-maven-remote

**trial-libs-release** - trial-libs-release-local + trial-maven-remote

In the next project we will build a jenkins pipeline to build and store artifact using Jfrog Artifactory.

Demo maven project with Jfrog + jenkins pipeline integration.