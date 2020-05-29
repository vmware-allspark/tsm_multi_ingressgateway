# tsm_multi_ingressgateway
Manifest to deploy additional ingress gateways in Tanzu Service Mesh managed cluster. 

1) Onboard a client K8s cluster on Tanzu Service Mesh (TSM). 
2) Once onboarded TSM will install and manage the lifecycle of Istio on the onboarded Cluster. 
3) By default TSM creates one istio ingressgateway deployment per cluster, however for some usecases Customers might want multiple istio ingressgateway. 
4) For scenarios where second ingressgateways are needed, peform "kubectl apply -f ingressgateway-manifest/second-ingressgateway.yaml" to deploy a second ingressgateway. 
5) For more than two ingressgateways, replace "second" in the yaml with different prefix to add more ingressgateways. 
6) For attaching gateway or other objects to the second-ingressgateway, use the selector "istio: second-ingressgateway". Refer to example.

# GNS and multiple ingress gateways

In the following section lets explore a usecase where we have two multi-cluster applications deployed in their own namespaces and using one ingressgateway per application to access the frontend microservice of the application. 

1) Onboard two clusters to TSM. 
2) Lets split bookinfo app and install all the apps except details in cluster-1 bookinfo namespace and install details on cluster-2 bookinfo namespace. 

  - On cluster-1 
     ```kubectl label ns bookinfo istio-injection=enabled
        kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/platform/kube/bookinfo.yaml -l app!=details,account!=details -n bookinfo
     ```
  
  - On cluster-2 
     ```kubectl label ns bookinfo istio-injection=enabled
        kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/platform/kube/bookinfo.yaml -l app=details -n bookinfo
        kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/platform/kube/bookinfo.yaml -l account=detail -n bookinfo
      ```
3) Deploy gateway and virtualservice for accessing bookinfo through through the default ingressgateway deployed by TSM. Apply the following on cluster-1

    ``` 
    kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/networking/bookinfo-gateway.yaml -n bookinfo
    
    ```
4) Check if you are able to access the bookinfo app through the ingressgateway:

    ``` 
    $ kubectl get svc -n istio-system istio-ingressgateway
    NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)                                                                                                                      AGE
    istio-ingressgateway   LoadBalancer   100.71.100.126   aa3a9e4489d1d11eaada402d26df238b-2032527493.us-west-2.elb.amazonaws.com   15020:31573/TCP,80:32316/TCP,443:31686/TCP,31400:30371/TCP,15029:30374/TCP,15030:30828/TCP,15031:30262/TCP,15032:32554/TCP   5d9h
    
    $ curl -s "http://aa3a9e4489d1d11eaada402d26df238b-2032527493.us-west-2.elb.amazonaws.com/productpage" | grep -o "<title>.*</title>"
    <title>Simple Bookstore App</title>
    
    ```

    


4) Lets split acme app and install all the services except catalog in cluster-1 in default namespace and install catalog service in cluster-2 default namespace.

  - Clone Acme app:
     ``` 
     git clone  -b dkalani-dev3    https://github.com/vmwarecloudadvocacy/acme_fitness_demo.git
     
    ```

  - On cluster-1
     ``` kubectl label ns default istio-injection=enabled
         kubectl apply -f acme_fitness_demo/kubernetes-manifests/secrets.yaml
         kubectl apply -f acme_fitness_demo/istio-manifests/gateway.yaml
         kubectl apply -f acme_fitness_demo/kubernetes-manifests/acme_fitness_cluster1.yaml
    ```
  - On cluster-2
    ``` kubectl label ns default istio-injection=enabled
        kubectl apply -f acme_fitness_demo/kubernetes-manifests/secrets.yaml
        kubectl apply -f acme_fitness_demo/kubernetes-manifests/acme_fitness_cluster2.yaml
     ```
  
  5) Create second-ingressgateway on cluster-1:
      
    ``` 
        kubectl apply -f ingressgateway-manifest/second-ingressgateway.yaml
         
     ```
  6) Create gateway and virtual-service for accessing acme app. Attach the gateway to "second-ingressgateway"
  
      ``` 
        Edit acme_fitness_demo/istio-manifests/gateway.yaml and change 
        
        selector:
        istio: ingressgateway 
        
        to
        
        selector:
        istio: second-ingressgateway 
        
        kubectl apply -f acme_fitness_demo/istio-manifests/gateway.yaml
           
     ```
     
   7) Check if you are able to access ACME app through the second-ingressgateway
   
         ``` 
          $ kubectl get svc -n istio-system second-istio-ingressgateway
          NAME                          TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                                                                                                                      AGE
         second-istio-ingressgateway   LoadBalancer   100.65.128.188   a9cc4586ea0f611eaada402d26df238b-823012376.us-west-2.elb.amazonaws.com   15020:32170/TCP,80:32388/TCP,443:31142/TCP,31400:31164/TCP,15029:30173/TCP,15030:31063/TCP,15031:32682/TCP,15032:30968/TCP   11h
         $ curl -s "http://a9cc4586ea0f611eaada402d26df238b-823012376.us-west-2.elb.amazonaws.com/" | grep -o "<title>.*</title>"
         <title>ACME Fitness</title>
           
       ```
       
   8) Now that both the applications are accesible through their own ingressgateways, we need to Create two GNS in TSM to make cross-cluster communication work the applicatins. 
   
   9) Create "acme" GNS and include the "default" namespace on both the clusters. We are using "demo.acme.com" as GNS domain name. So edit the shopping deployment on default namespace of Cluster-1 and change the environment variable CATALOG_HOST to catalog.demo.acme.com.
   
   10) Create "bookinfo" GNS and include the "bookinfo" namespace on both the clusters. We are using "demo.bookinfo.com" as GNS domain name. So edit the productpage deployment on bookinfo namespace of Cluster-1 and change the add environment variable.
   
   
   

  
  
  
  
  
      
  
  
  
  
  
  
  



