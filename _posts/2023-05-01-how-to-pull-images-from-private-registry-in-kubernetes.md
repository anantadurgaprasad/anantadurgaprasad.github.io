---
layout: post
title:  "How to pull images from private registry — Jfrog/ECR in kubernetes."
categories: kubernetes
tags: ecr jfrog
---
{% include toc %}
while I was practicing kubernetes I got a doubt I’m pulling images from public registry but what if I want to pull image from private registry.

### How to push image to private registry

I learnt about how to push to private registry while working with docker. I will briefly talk about the steps here.  
make sure you are logged in already.

1.  Get the servername/URI of the private registry

```
durga.dkr.ecr.ap-south-1.amazonaws.com/demo   
<accountid/alias>.dkr.ecr.<aws-region>.amazonaws.com/repo_name
```

2. build the image using the below command using dockerfile

```
docker image build -t image_name:version . 
```

you can check the above built image doesn’t have any repository with the command

```
docker image ls
```

3. Retag the image with the repository you want to push

```
docker tag image_name:version durga.dkr.ecr.ap-south-1.amazonaws.com/demo:latest
```

4. now push the image to the private registry

```
docker push durga.dkr.ecr.ap-south-1.amazonaws.com/demo:latest
```

### Pulling Images using ImagePullSecrets and Secret k8 object:

For this example, I created a private ECR in AWS. I pushed an image to it as well using above method specified.

In order to pull image from private registry we need to authenticate against the registry . so we need to get credentials and create secret k8 object with all the credentials.

```
kubectl create secret -n <pod_namespace> docker-registry <secret_name>   
  --docker-server=DOCKER_REGISTRY_SERVER   
  --docker-username=DOCKER_USER   
  --docker-password=DOCKER_PASSWORD 
```

Once the secret is created you need to edit the pod/deployment.yaml and add the attribute — _imagePullSecrets_

```
apiVersion: v1  
kind: Pod  
metadata:  
  name: foo  
  namespace: <pod_namespace>  
spec:  
  containers:  
    - name: foo  
      image: durga.dkr.ecr.ap-south-1.amazonaws.com/demo:latest  
  imagePullSecrets:  
    - name: <secret_name>
```

```Note — The secret must be in the same namespace as pod/deployment for the k8 object to use the secret.```

If we don’t want to add the secret everytime we create a new pod or if we want to use new secret we have to update in all the pod and deployment k8 objects. So, instead we can attach this to a default service account . As the default service account is attached by default to every pod/deployment. we don’t have to attach it everytime .

### Securing Access to Images
If there is sensitive data or something and you don’t want your image to available to other pods in the cluster. Another case is if the cluster multi tenant and you want to secure your image you can use below link to do so.

[https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#alwayspullimages)  
I will be writing another article about the above use case and updating this one soon.

All the above things work the same with jfrog. you can use access token to login to jfrog and to pull the images.