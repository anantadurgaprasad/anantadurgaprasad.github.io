---
title: "Working with Helm Charts"
layout: post
tags: helm   
---
### Introduction to Helm Charts
Helm is a powerful tool used in the Kubernetes ecosystem to streamline the installation and management of applications. Helm introduces the concept of "charts," which are packages of pre-configured Kubernetes resources.
 


In this Article We will discuss
1. What are Helm Charts ?
2. How to start writing Helm Charts ? 
3. Syntax for writing Helm Charts
 
This comprehensive guide will help you with the knowledge to effectively utilize Helm charts in your Kubernetes deployments.

### What are Helm Charts ? 

The official website describes [HelmChart](https://helm.sh/) as ```The
package manager
for Kubernetes```.

Helm charts are essentially packages containing all the necessary files to describe a related set of Kubernetes resources.
A single chart might be used to deploy something simple, like a stateless web application, or something complex, like a full web app stack with HTTP servers, databases, caches, and so on.

#### Components of Helm Charts:

```Chart.yaml:``` A YAML file containing metadata about the chart.

```Values.yaml:``` A YAML file that specifies the default configuration values for the chart.

```Templates/:``` A directory containing template files that generate Kubernetes manifest files when combined with values.

```Charts/:``` An optional directory containing any charts upon which this chart depends.

```README.md:``` An optional README file with instructions for using the chart.


### How to Start Writing Helm Charts ?

To begin writing Helm charts, you will need to set up your development environment and understand the basic components of a Helm chart. Here’s how you can get started:

Setting Up Your Environment:

```Install Helm:``` First, ensure that Helm is installed on your system. You can follow instructions from [Helm official documentation](https://helm.sh/docs/intro/install/).

```Set Up Kubernetes:``` You can use a local setup like Minikube or a managed Kubernetes service. Helm charts will be deployed to this Kubernetes environment.

```Initialize Helm:``` Run ```helm init``` to set up Helm on your local machine and set up Tiller in your Kubernetes cluster.

**Creating Your First Chart:**

```Create a New Chart:``` Use the command ```helm create <chart-name> ```to create a new chart directory along with the necessary files.

```Customize Chart.yaml:``` Update the Chart.yaml file with details about your chart like name, version, and description.

```Define Default Values:``` Edit the values.yaml file to set default values for your chart.

```Develop Templates:``` Modify templates in the templates/ directory to define Kubernetes resources like pods, services, or ingresses.


### Syntax for Writing Helm Charts

Writing Helm charts involves understanding the syntax used in the templates/ files. These templates are written in Go template language with some added Helm template functions.

**Basic Syntax Elements:**

*Values Replacement:* Use {% raw %}```{{ .Values.key }}```{% endraw %} to inject values from your values.yaml file.

*Control Structures:* You can use control structures from Go templates such as 
*if-else block:*
{% raw %}```{{ if .Values.enabled }}...{{ end }}```{% endraw %} to include or exclude parts of the template based on values.

{% raw %}```{{- range $key, $params := .Values.ssm.params }}....{{ end }}```{% endraw %} for looping through an array or map of values 


*Functions and Pipelines:* Helm provides many functions like default and quote, and you can use pipelines to modify values,
 e.g.,{% raw %} ```{{ .Values.name | upper }}.```{% endraw %}


Below is the one of the template file of a helm chart which has conditional if-else, range loop and retrieve value by index
{% raw %}
```
{{- if .Values.ssm.enabled -}}  # conditional if block
{{- range $key, $params := .Values.ssm.params }} # range loop block
---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: {{ $key }}-store # variable injection
  namespace: portals
spec:
  refreshInterval: {{ $.Values.refreshInterval }} # variable injection
  secretStoreRef:
    name: secretstore-awssm
    kind: SecretStore
  target:
    name: {{ $key }}-secret
    creationPolicy: Owner
  data:
    {{- range $paramKey, $paramVal := index $.Values.ssm.params $key }} # index
    - secretKey: {{ $paramKey }}
      remoteRef:
        key: "/{{ $.Values.prefix }}/{{ $.Values.env }}/{{ $paramVal }}"
    {{- end }}
{{- end }}
{{- end}}


```
{% endraw %}

**Validating Template**:
Once the helm chart is developed along with values.yml . we can validate helm chart by running following commands 
```
helm lint PATH [flags] 

helm template [NAME] [CHART] [flags] 

#you can learn more about the commands by running 
helm lint --help or helm template --help
```
**Deploying Helm Chart:**
Below command is used to deploy a helm chart to kubernetes 
```helm upgrade [RELEASE_NAME] [CHART] --install --namespace [NAMESPACE]
```

**Please refer below github repo link that has a helm chart I wrote and talks about how to use it. Tinker with it and  happy helming!**

[Helm chart Github Repo Link](https://github.com/anantadurgaprasad/helm.git)

**Conclusion:**
Helm charts provide a robust method for managing and deploying applications in Kubernetes environments. By understanding how to write and use Helm charts, you can greatly simplify deployment processes and ensure consistency across environments. 

With the basic knowledge of chart creation and syntax provided in this guide, you are now equipped to start developing your own Helm charts and exploring further customization and optimization possibilities.