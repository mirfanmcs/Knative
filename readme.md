# Serving 

Install
--------
Istio should be installed before installing Knative. 

Run following command to install Knative on minikube:

`$ kubectl apply -f https://github.com/knative/serving/releases/download/v0.2.2/release-lite.yaml`


Enable automatic inject of Envoy proxy in default namespace
-------------------------------------------------------------
`$ kubectl label namespace default istio-injection=enabled`

Deploy service 
--------------
`$ kubectl apply -f serving/customer-service.yaml`

Call the service 
-------------------
Get host of service:

`$ kubectl get services.serving.knative.dev customer  --output=custom-columns=NAME:.metadata.name,DOMAIN:.status.domain`

Get the NodePort/Loadbalancer port of knative-ingressgateway:

`$ kubectl get svc knative-ingressgateway --namespace istio-system`

Get ip address of cluster:

`$ minikube ip`

Call service: 

`$ curl -H "Host: customer.default.example.com" http://192.168.99.100:32380/123`

View Resources
---------------
To view services:

`$ kubectl get services`

To view routes:

`$ kubectl get routes`

To veiw configurations:

`$ kubectl get configurations`

To view revisions:

`$ kubectl get revisions`

Blue Green Deployment
----------------------
Deploy v1 (blue) of customer service:

`$ kubectl apply -f serving/blue-green/config-v1.yaml`

Deploy route of v1:

`$ kubectl apply -f serving/blue-green/route-v1.yaml`

Run service, all traffic will route to v1 (blue).

Deploy v2 (green) of customer service:

`$ kubectl apply -f serving/blue-green/config-v2.yaml`

This will create another revision of customer service. 

To see all revisions run following command:

`$ kubectl get revisions`

This will show following revisions:

customer-00001   
customer-00002  

To send 90% traffic to v1 and 10% to v2 apply following route:

`$ kubectl apply -f serving/blue-green/route-90-10.yaml`

To increase load of 50% to v2, apply following route:

`$ kubectl apply -f serving/blue-green/route-50-50.yaml`

When ready, route 100% traffic to v2 (green):

`$ kubectl apply -f serving/blue-green/route-v2-all.yaml`


Routing
------------
Create a VirtualService with host "example.com", and define routing rules in the VirtualService so that "example.com/customer" maps to the Customer service, and "example.com/transaction" maps to the Transaction service.


Deploy transaction service:

`$ kubectl apply -f serving/transaction-service.yaml`

Deploy the route: 

`$ kubectl apply -f serving/routing/routing.yaml`

Call Customer service:

`$ curl -H "Host: example.com" http://192.168.99.100:32380/customer/123`

Call Transaction service:

`$ curl -H "Host: example.com" http://192.168.99.100:32380/transaction/123`


Cleanup
---------
`$ kubectl delete -f serving/customer-service.yaml`

`$ kubectl delete -f serving/blue-green/config-v1.yaml`

`$ kubectl delete -f serving/blue-green/config-v2.yaml`

`$ kubectl delete -f serving/blue-green/route-v2-all.yaml`

`$ kubectl delete -f serving/transaction-service.yaml`

`$ kubectl delete -f serving/routing/routing.yaml`



# Build

Install Knative build
------------------------
You should have Knative installed already. 

`$ kubectl apply -f https://storage.googleapis.com/knative-releases/build/latest/release.yaml`


## Using Kaniko Build Template

Install kaniko build template
--------------------------------

`$ kubectl apply -f https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml`

To see installed buildpacks: 

`$ kubectl get buildtemplates`


Register secrets for Docker Hub
---------------------------------

On macOS or Lunux, run following command to get base 64 encoded username and password for secret:

`$ echo -n "mysecret" | base64`

Create secret using the docker-secret.yaml file:

 `$ kubectl apply -f build/docker-secret.yaml` 

Create a new Service Account
-------------------------------
 Create a new Service Account manifest which is used to link the build process to the secret.

 `$ kubectl apply -f build/docker-service-account.yaml`

Deploy the transaction service build and service:

`$ kubectl apply -f build/transaction-service-build.yaml`

To check the state of the service:

`$ kubectl get ksvc -o yaml`

Call the service 
-------------------
Get host of service:

`$ kubectl get services.serving.knative.dev transaction  --output=custom-columns=NAME:.metadata.name,DOMAIN:.status.domain`

Get the NodePort/Loadbalancer port of knative-ingressgateway:

`$ kubectl get svc knative-ingressgateway --namespace istio-system`

Get ip address of cluster:

`$ minikube ip`

Call service: 

`$ curl -H "Host: transaction.default.example.com" http://192.168.99.100:32380/123`

Cleanup
---------

`$ kubectl delete -f build/docker-secret.yaml` 

`$ kubectl delete -f build/docker-service-account.yaml`

`$ kubectl delete -f build/transaction-service-build.yaml`


# Eventing 

Install Knative Eventing
--------------------------
You should have Knative Serving installed already.

Install the core Knative Eventing (which provides an in-memory ChannelProvisioner) and the core sources:

`$ kubectl apply -f https://github.com/knative/eventing/releases/download/v0.3.0/release.yaml`

`$ kubectl apply -f https://github.com/knative/eventing-sources/releases/download/v0.3.0/release.yaml`

Saee following link for Knative eventing samples:

https://github.com/knative/docs/tree/master/eventing/samples



# Project riff

Project riff makes it easy to work with Knative (serving, build, eventing). With riff CLI you can create Knative serving, build, and eventing. CLI inteact with Knative directly without the need for you to create yaml files for configuration. 

Visit following link for project riff home page and introductory videos: https://projectriff.io

Visit following link for documentation: https://projectriff.io/docs/getting-started

Visit following link for blogs: https://projectriff.io/blog/


If you are using Spring Boot, create riff function using Spring Cloud Function https://cloud.spring.io/spring-cloud-function/.
