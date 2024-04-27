---
title: "Maven project with Jfrog Artifactory + jenkins pipeline integration"
layout: post
categories: jfrog
tags: jfrog  
---

If you are new to JFrog Artifactory , please read the previous post to understand the fundamentals

[Fundamentals of JFrog](https://medium.com/@anantadurgaprasadar/fundamentals-of-jfrog-7a6b0e996835)

Content :

1.  Download Jenkins Artifactory Plugin.
2.  Configuring Artifactory Plugin
3.  Creating Jenkins Pipeline.

1. Download the Jenkins Artifactory Plugin :
---------------------------------------------

From the Jenkins dashboard

go to manage jenkins > manage plugins > search for **Artifactory in available** and click install without restart.

I already installed it so I’m showing in installed section.

**2.Configuring Artifactory Plugin**

Go to mange jenkins → configure system and search for jfrog . If you see something like below ==> jfrog is installed successfully.

**Give the instance id — any name as you like.**

**Give the platform url that you created.**

**Provide the credentials either directly here or through manage credentials.**

**click on test connectivity to check the jenkins is able to reach jfrog.**

**3. Creating Jenkins Pipeline**

There are two types of pipeline w.r.t to jfrog.

1.  Where pipeline uploads artifact to jfrog repository we created.
2.  Where pipeline uses artifact in jfrog to build another project.

**Upload Pipeline**

We create jenkins job in the usual way.we use Artifactory plugin installed to use jfrog in pipeline

Useful Links :

[Jenkins plugin Documentation](https://plugins.jenkins.io/artifactory/?source=post_page-----71ee77d812e3--------------------------------)
[jenkinsfile examples](https://github.com/jfrog/project-examples/tree/master/jenkins-examples/pipeline-examples/declarative-examples?source=post_page-----71ee77d812e3--------------------------------)

The jfrog+jenkins pipeline is pretty standard and almost same as the one mentioned in the github examples.

```
pipeline {  
    agent any  
    stages {  
        stage ('Clone') {  
            steps {  
                git branch: 'master', url: "https://github.com/jfrog/project-examples.git"  
            }  
        }  
  
        stage ('Artifactory configuration') {  
            steps {  
                rtServer (  
                    id: "ARTIFACTORY_SERVER",  
                    url: SERVER_URL,  
                    credentialsId: CREDENTIALS  
                )  
  
                rtMavenDeployer (  
                    id: "MAVEN_DEPLOYER",  
                    serverId: "ARTIFACTORY_SERVER",  
                    releaseRepo: ARTIFACTORY_LOCAL_RELEASE_REPO,  
                    snapshotRepo: ARTIFACTORY_LOCAL_SNAPSHOT_REPO  
                )  
  
                rtMavenResolver (  
                    id: "MAVEN_RESOLVER",  
                    serverId: "ARTIFACTORY_SERVER",  
                    releaseRepo: ARTIFACTORY_VIRTUAL_RELEASE_REPO,  
                    snapshotRepo: ARTIFACTORY_VIRTUAL_SNAPSHOT_REPO  
                )  
            }  
        }  
  
        stage ('Exec Maven') {  
            steps {  
                rtMavenRun (  
                    tool: MAVEN_TOOL, // Tool name from Jenkins configuration  
                    pom: 'maven-examples/maven-example/pom.xml',  
                    goals: 'clean install',  
                    deployerId: "MAVEN_DEPLOYER",  
                    resolverId: "MAVEN_RESOLVER"  
                )  
            }  
        }  
  
        stage ('Publish build info') {  
            steps {  
                rtPublishBuildInfo (  
                    serverId: "ARTIFACTORY_SERVER"  
                )  
            }  
        }  
    }  
}
```

You can see the artifactory plugin documentation to see what fields are necessary and what are optional.

In our case from the repo we created in previous post :

**rtMavenResolver**

releaserepo → trial-libs-release

snapshotrepo → trial-libs-snapshot

**rtMavenDeployer**

releaserepo → trial-libs-release-local

snapshotrepo → trial-libs-snapshot-local

**rtMavenRun**

**By default artifactory deploys your artifact to jfrog. you don’t have to add deploy cmd in goal section.**

**Resolver** → when maven build takes place all the dependencies are resolved through the repos mentioned in the resolver block in the pipeline script.

By default virtual repo has local repo and remote repo .

The repos mentioned in Resolver block is a virtual repo = **you can add the local repos which hold the artifacts that are dependencies for the project your building .**

**Deployer** → when maven artifact is produced it will be uploaded to the repo mentioned in the deployerId block.

Based on the project is a snapshot or release the respective repos will be used.

The publishBuildInfo block is optional — you can keep it if you want build info else you can remove it.

```
pipeline {  
    agent any  
    tools {  
        maven "MAVEN_HOME"  
    }  
    stages {  
        stage ('Clone') {  
            steps {  
                git branch: 'master', url: "https://github.com/anantadurgaprasad/web_ex.git"  
            }  
        }  
  
        stage ('Artifactory configuration') {  
            steps {  
                // rtServer (  
                //     id: "ARTIFACTORY_SERVER",  
                //     url: SERVER_URL,  
                //     credentialsId: 'jfrog_creds'  
                // ) I don't need this step as I configured the jfrog from mangaeJenkins>globalconfiguration>jfrog  
  
                rtMavenDeployer (  
                    id: "MAVEN_DEPLOYER",  
                    serverId: "jfrog-server-1", //name you gave while configuring jfrog in gloabal settings.  
                    releaseRepo: "trial-libs-release-local",  
                    snapshotRepo: "trial-libs-snapshot-local"  
                )  
  
                rtMavenResolver (  
                    id: "MAVEN_RESOLVER",  
                    serverId: "jfrog-server-1",  
                    releaseRepo: 'trial-libs-release',  
                    snapshotRepo: 'trial-libs-snapshot'  
                )  
            }  
        }  
  
        stage ('Exec Maven') {  
            steps {  
                rtMavenRun (  
                    tool: 'MAVEN_HOME', // Tool name from Jenkins configuration  
                    pom: 'pom.xml',  
                    goals: 'clean install -U', //you can specify any flags as in normal maven commands  
                    deployerId: "MAVEN_DEPLOYER",  
                    resolverId: "MAVEN_RESOLVER"  
                )  
            }  
        }  
  
        stage ('Publish build info') {  
            steps {  
                rtPublishBuildInfo (  
                    serverId: "jfrog-server-1"  
                )  
            }  
        }  
    }  
}
```

Once the pipeline is successful you can see the artifact in the jfrog.

**The folder structure in the repo is a default structure that is used by jfrog for that particular type of package . you can create your own folder structure as well.**

**As the project is snapshot.Even if the code is same and version is same we have one artifact for each time I ran pipeline. They are differentiated using timestamp.**

**But for release repo there will be only one artifact for each version.**

For pipelines which use jfrog artifacts to build projects. We need to pass a customised settings.xml as a parameter.

```
mvn -s path/to/settings.xml clean install
```

How to producing customised settings.xml is mentioned in below article.

Next Article :

Configuring other projects to use artifact from jfrog.