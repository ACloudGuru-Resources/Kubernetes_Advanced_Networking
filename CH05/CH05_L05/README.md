# Lab Name

### Objectives

### Prereq

### Setup Steps



Create a mesh with the create-mesh command. 
aws appmesh create-mesh --mesh-name apps

Create a virtual service with the create-virtual-service command. 

aws appmesh create-virtual-service --mesh-name apps --virtual-service-name serviceb.apps.local --spec {}

Create the virtual node with the create-virtual-node command using the JSON file as input. 

aws appmesh create-virtual-node --cli-input-json file://virtual-node-ping.json

Create the virtual router with the create-virtual-router command using the JSON file as input. 

Create the route with the create-route command using the JSON file as input. 
aws appmesh create-virtual-router --cli-input-json file://create-virtual-router.json


aws appmesh create-route --cli-input-json file://create-route-ping.json