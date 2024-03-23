---
title: "Fundamentals of Transit Gateway"
layout: post
categories: aws networking
tags: transitgateway  
---

### Introduction to AWS Transit Gateway
Amazon Web Services (AWS) offers a plethora of tools designed to streamline and optimize the digital infrastructure of businesses, among which the AWS Transit Gateway stands out as a pivotal solution. ```This service acts as a network transit hub, enabling the interconnection of VPCs, AWS accounts, and on-premises networks through a central gateway. By facilitating this unified point of connectivity, AWS Transit Gateway simplifies network management, reduces operational complexity, and enhances the overall scalability of the network architecture.```

### Understanding the Use of Transit Gateway

The Transit Gateway is instrumental in architecting a simplified, more manageable network structure. It allows organizations to connect multiple Virtual Private Clouds (VPCs) and other networks without the need to establish individual, pairwise connections. This centralized network hub not only streamlines connectivity but also improves the efficiency of data transfer across the network. 

Transit Gateway mains Consists of :
1. Transit Gateway Attachment 
2. Transit Gateway Route Table


**Transit Gateway Attachments** :
This feature enables the connection of various network entities, such as Virtual Private Clouds (VPCs), VPN connections, and AWS Direct Connect gateways, to the Transit Gateway. ```**Essentially, an attachment represents the linkage between the Transit Gateway and a network entity, serving as the conduit through which traffic flows into and out of the Transit Gateway.**```

#### Types of Attachments:

**VPC Attachments**: Allow VPCs to connect to the Transit Gateway, facilitating inter-VPC communication and access to shared services.

**VPN Attachments**: Enable the connection of on-premises networks to the Transit Gateway via VPN, extending the corporate network into the cloud.

**Direct Connect Gateway Attachments**: Connect the Transit Gateway to an AWS Direct Connect gateway, offering a private, dedicated network connection to AWS.

**Peering Attachments**: Support the peering of two Transit Gateways, allowing for the routing of traffic between different regions or AWS accounts.

![ss](/images/tgw_diagram.jpeg)

#### Transit Gateway Route Tables Explained
AWS Transit Gateway route tables play a critical role in directing traffic between the connected entities (VPCs, AWS accounts, on-premises networks). These tables hold the rules that determine the path network traffic should take, essentially guiding data packets to their intended destinations. By configuring route tables, administrators can manage and optimize the flow of traffic

```**For Every TransitGateway Attachment we can have a Route Table. Routing of all the traffic Directed toward that attachment is determined by the associated Route Table**```

**Route Propagation in Transit Gateway**
Route propagation in AWS Transit Gateway refers to the automatic updating of route tables when new routes are added.  The dynamic nature of route propagation minimizes the administrative overhead associated with manual route updates, making network management more efficient and error-prone. 


**One of the most know architectures with Transit Gateway is  2 Transit Gateways with peering attachment between them allowing the VPCs on either side of Transit Gateway to communicate.**

#### To create this architecture follow the below steps : 

**prerequistites :**

Note - Since we don't have multiple aws accounts we will create 2 transit gateways within the same aws account. I created both of them in the same region for convience but they can be any accounts. 

2 VPC's each with a route table and associated subnets. 

*  Create 2 transit gateways which mirror the 2 gateways we have in architecture 
``` project1 vpc is associated with transitgateway region1 and project2 vpc with transitgateway region1```

**Note - As a Best Practice it is always suggested to disable route table association and route table propagation when creating Transit Gateway . This ensure granular control.**

*  Create Transit Gateway VPC attachment with each VPC and their respective transit gateways.


*  Create a Transit Gateway peering attachment betweent the transit gateways. TO achieve this one of the transit gateway has to request attachment with another transit gateway , only if the other gateway accepts the attachment your attachment will be available else it will be in pending state.

* As we disabled auto route table association , A route table has to be created for each of the attachment. This route table will dictate routing of traffic directed at the attachment. 

**Note - We can associate one route table with any number of attachment but it's a best practice to associate one route table for each of the attachment.**

 * Creating Route Table for each VPC attachment and for each of the peering attachment 

* Updating the Route Tables of the VPC to direct traffic to transit gateway attachment 

[Scribe Link](https://scribehow.com/shared/Create_Transit_Gateway_with_Peering_between_Regions__S3WIwG37QjyHfSM2fWV7Kw)