---
title: "Jenkins+Jfrog for managing Npm packages"
layout: post
categories: jfrog
tags: jfrog  
---

In this post, I’m going to discuss about jenkins pipeline script for jfrog.

First, I will discuss the correct process to implement and then talk about the errors I came across.

There are two pipelines for npm+jfrog integration

1.  Upload Pipeline — uploading npm package to jfrog
2.  Download Pipeline — Pipeline for using npm package in jfrog
3.  Upload Pipeline :

I used .npmrc file to make necessary configurations for uploading npm package to jfrog.

This configuration file can be downloaded from the jfrog artifactory itself.

Create a npm repo > click on it and click on set me up

We can insert the password and click insert. The encrypted password will be inserted.

npm config set registry is not feasible for implementing in jenkins server as the npm configurations may differ from pipeline to pipeline.

so we will use npmrc file to do the configurations.

Copy and paste the file in npmrc file in the pipeline.

The pipeline code is

```
pipeline{  
  agent any  
  tools {  
    nodejs "NODEJS"  
     
  }  
  stages{  
    stage('Clean Workspace'){  
      steps{  
        sh 'ls'  
        cleanWs deleteDirs: true  
        sh 'ls'  
      }  
    }  
      
    stage('git checkout'){  
          
      steps{  
        git branch: '1.0.0', credentialsId: 'bit_bucket_creds', url: ''  
      }  
    }  
    stage('npm install and build') {  
      
        steps{  
              
                  
                sh 'cp /var/lib/jenkins/workspace/.npmrc .'  
                  
                sh 'cat .npmrc'  
                  
                sh 'ls -al'  
                sh "npm install"  
                sh "npm run release -- --first-release"  
                sh "npm run build"  
                sh 'ls -al '  
                  
                sh 'pwd'  
                  
  
        }  
      
    }  
      
    stage('Npm publish'){  
      
    steps{  
          
        sh 'npm publish'  
    }  
          
    }  
      
  }  
}
```

Download Pipeline :

I had issues with the download pipeline because we want to use access token instead of password. I tried replacing password with access token but it gave me error.

After a quite a trial and error. I came to know that we should put encrypted (base 64) access token in place of password.

So the download pipeline is very simple. Just put the npmrc file in the same directory as the github code.

In order to get the configurations for the npmrc file it’s very simple using curl

Check the documentation here

[

npm Registry
------------

### Artifactory allows you to define any layout for your npm registries. In order to upload packages according to your…

www.jfrog.com

](https://www.jfrog.com/confluence/display/JFROG/npm+Registry?source=post_page-----449d784d5877--------------------------------)

I primarily took long time because of the following reasons

1.  I couldn’t find the particular page of documentation initially . so I tried other solutions from internet
2.  Even though I finally got the configuration right. The build failed because the checksum was different to that in package-lock.json file. I thought it was issue with npmrc file. So I kept changing it.( the pipeline logs are insufficient please check the logs in the log-file mentioned)
3.  When i removed the package-lock.json file and then built in the build still failed. It was because one of the dependency was not mentioned in the package.json file. ( this was resolved easily anyway)

These are the errors I faced and the reason behind those errors. Make sure you check all these to understand your error.