---
title: "User Management in Jfrog"
layout: post
categories: jfrog
tags: jfrog  
---


In this section we discuss about different features that jfrog provides to manage access to artifacts.

There are 5 sub-sections in User Management section

1.  Users
2.  Groups
3.  Global Roles
4.  Permissions
5.  Access Token

In this we are going to discuss Permissions and Access Tokens. Remaining are very general and intuitive. Permissions and some aspects of Access Tokens are unique to Jfrog and takes a bit to understand.

**Permissions :**

It defines access to a subset of resources in jfrog — repos/builds and the actions that a user/group assigned that permission can perform.

1.  Give a name and click on the repos and builds that you want to give user access to.

2. Select a user or a group and specify the set of actions they can perform on the repos and then click create.

**Access Token :**

Once you create a user and assign it permissions. we want to give that same level of permissions to multiple users. This is where Access Token comes in

In a company the developers might need access to jfrog to read the artifacts they might need while developing the project and building it..

Instead of sharing credentials. We can create access token and share it with all the dev team.

**Advantages :**

The tokens can’t be used to login to jfrog ui.

The token can be revoked whenever we want.

We can change the permissions of that token user and all the people using it get the new permissions.

By checking “Create Reference Token” we will get a smaller size access token.

We can choose the token scope . if we chose a user then the permissions of that user will be applied to that token.

**Refreshable Access Token** — These tokens are very useful when you want to extend access token based on necessity. These are useful if you have subscription model and want to extend user access based on subscription.

These tokens can be created through REST API’s provided by jfrog.

We can create token in this way and can put that in the settings.xml to use jfrog to resolve dependencies.

Rest api documentation — [https://www.jfrog.com/confluence/display/JFROG/Artifactory+REST+API](https://www.jfrog.com/confluence/display/JFROG/Artifactory+REST+API)

In the next article we will discuss REST apis of jfrog.