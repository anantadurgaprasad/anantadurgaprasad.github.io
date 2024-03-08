---
title: "Configuring other projects to use artifact from jfrog"
layout: post
categories: jfrog
tags: jfrog  
---


We use jfrog Artifactory in general to store artifacts which can be used later for building other projects instead of building every time we build other projects.

Configuring projects is very straight forward.

1.  Generating Settings.xml file
2.  Using that settings.xml file while building the artifact.
3.  Generating Settings.xml file

In almost all the sources I saw you simply click on repo and click set me up.

There you can find all the instructions which are easy to understand and use.

But this is not the usecase I want to address in this post.

**In this post the use case is if you have multiple artifacts in different repos in jfrog and all those repos are dependencies for another project you are building then you can use the below approach**

Create a virtual repo which has all the repos in it that you need to build the project.

Here you can include one remote repo if you want to resolve all the dependencies from jfrog and want jfrog Artifactory to act as proxy for building the project.

But if you want to resolve only the dependencies that you uploaded to jfrog and remaining all from maven central repo . Don’t include remote repo.

Check the settings.xml that I provide for the above use case.

Now, once you create virtual repo _click on that repo > click set me up_

I have selected the trial-libs-release-local and trial-libs-snapshot-local repos. you can select any repo you want.

Note: you can only select the repos with the same package type as the virtual repo being created.

_select the virtual repo you created in all the options , insert the password and click generate settings > download snippet_

The generated settings.xml has to be edited to remove redundancy and also for resolving the other dependencies from central maven repo.

Below is the final settings.xml

```
<?xml version="1.0" encoding="UTF-8"?>  
<settings xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 http://maven.apache.org/xsd/settings-1.2.0.xsd" xmlns="http://maven.apache.org/SETTINGS/1.2.0"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">  
  <servers>  
    <server>  
      <username><you will find username here></username>  
      <password><encrypted password here ></password>  
      <id>jfrog-repo</id>  
    </server>  
  </servers>  
  <profiles>  
    <profile>  
      <repositories>  
        <repository>  
     <id>central</id>  
     <url>https://repo.maven.apache.org/maven2/</url>  
    </repository>  
        <repository>  
<!-- I changed the id here -- !>  
          <id>jfrog-repo</id>   
          <name>trial-dependency-artifacts-repo</name>  
          <url>https://**.jfrog.io/artifactory/trial-dependency-artifacts-repo</url>  
        </repository>  
      </repositories>  
        
      <id>artifactory</id>  
    </profile>  
  </profiles>  
  <activeProfiles>  
    <activeProfile>artifactory</activeProfile>  
  </activeProfiles>  
</settings>
```

I struggled to get to this settings.xml . With help from very good backend dev in the project.

**If you want all the dependencies to be resolved by jfrog , remove the central repo part and add the remote repo in the virtual repo you created.**

reference link :

[

How to make maven download dependencies from central repo. rather than downloading from remote…
-----------------------------------------------------------------------------------------------

### Thanks for contributing an answer to Stack Overflow! Please be sure to answer the question. Provide details and share…

stackoverflow.com

](https://stackoverflow.com/questions/70171935/how-to-make-maven-download-dependencies-from-central-repo-rather-than-downloadi?source=post_page-----ec5ed9f9f13--------------------------------)

> In the above approach the credentials are admin credentials. It is not a good practice. Instead create a user with only the necessary privileges and use access token instead of encrypted password so that you can revoke whenever you want.

In the next article we will discuss about User Management in Jfrog.

[

User Management in Jfrog
------------------------

### In this section we discuss about different features that jfrog provides to manage access to artifacts.

medium.com

](https://medium.com/@anantadurgaprasadar/user-management-in-jfrog-c7eab8679123?source=post_page-----ec5ed9f9f13--------------------------------)