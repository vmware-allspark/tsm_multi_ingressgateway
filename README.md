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

![Alt text](/images/two_clusters.png?raw=true)

2) Lets split bookinfo app and install all the apps except details in cluster-1 bookinfo namespace and install details on cluster-2 bookinfo namespace. 

  - On cluster-1 
     ```kubectl label ns bookinfo istio-injection=enabled
        kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/platform/kube/bookinfo.yaml -l app!=details,account!=details -n bookinfo
        
        $ kubectl get pods -n bookinfo --context=kaliappanm-prod-c1.k8s.local
        NAME                             READY   STATUS    RESTARTS   AGE
        productpage-v1-bcb4488d6-9swqt   2/2     Running   0          19h
        ratings-v1-7bdfd65ccc-9lhkf      2/2     Running   0          26h
        reviews-v1-5c5b7b9f8d-fg5fr      2/2     Running   0          26h
        reviews-v2-569796655b-7mlfr      2/2     Running   0          26h
        reviews-v3-844bc59d88-5rpsm      2/2     Running   0          26h
        
     ```
  
  - On cluster-2 
     ```kubectl label ns bookinfo istio-injection=enabled
        kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/platform/kube/bookinfo.yaml -l app=details -n bookinfo
        kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/platform/kube/bookinfo.yaml -l account=detail -n bookinfo
        
        $ kubectl get pods -n bookinfo --context=kaliappanm-prod-c2.k8s.local
        NAME                          READY   STATUS    RESTARTS   AGE
        details-v1-7f66769fdf-whw8w   2/2     Running   0          19h
        
      ```
3) Deploy gateway and virtualservice for accessing bookinfo through the default ingressgateway deployed by TSM. Apply the following on cluster-1

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
         kubectl apply -f acme_fitness_demo/kubernetes-manifests/acme_fitness_cluster1.yaml
         
         $ kubectl get pods -n default --context=kaliappanm-prod-c1.k8s.local
            NAME                              READY   STATUS    RESTARTS   AGE
            cart-6bf677c4d4-nzs2l             2/2     Running   0          9d
            cart-redis-56f4b69d98-v9vvr       2/2     Running   0          9d
            loadgenerator-6b4dc4555d-7d2qt    2/2     Running   0          9d
            order-54b4dbdf78-9lxbd            2/2     Running   0          9d
            order-mongo-5f5d88495b-hsffr      2/2     Running   0          9d
            payment-7c7f849c7c-f4jvj          2/2     Running   0          9d
            productpage-v1-77d9f9fcdf-8bg7z   2/2     Running   0          16h
            ratings-v1-7bdfd65ccc-lkbmz       2/2     Running   0          16h
            reviews-v1-6b7ddfc889-mkvnm       2/2     Running   0          16h
            reviews-v2-575b55477f-nmrnx       2/2     Running   0          16h
            reviews-v3-6584c5887c-nnzlx       2/2     Running   0          16h
            shopping-756b8d5d7b-78grx         2/2     Running   0          9d
            users-7c769bbf68-vjqkq            2/2     Running   0          9d
            users-mongo-cdf54b74d-snw68       2/2     Running   0          9d
    ```
  - On cluster-2
    ``` kubectl label ns default istio-injection=enabled
        kubectl apply -f acme_fitness_demo/kubernetes-manifests/secrets.yaml
        kubectl apply -f acme_fitness_demo/kubernetes-manifests/acme_fitness_cluster2.yaml
        
        $ kubectl get pods -n default --context=kaliappanm-prod-c2.k8s.local
          NAME                             READY   STATUS    RESTARTS   AGE
          catalog-5c589b5c5c-t7rt8         2/2     Running   0          24h
          catalog-mongo-5f5b55868d-gqzks   2/2     Running   0          24h
          loadgenerator-7dd5fcf578-pvpqc   2/2     Running   0          24h
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
   
   ![Alt text](/images/cluster1_view.png?raw=true)
   
   9) Create "acme" GNS and include the "default" namespace on both the clusters. We are using "demo.acme.com" as GNS domain name. 
   
   ![Alt text](/images/acme_gns.png?raw=true)
   
   10) Edit the shopping deployment on default namespace of Cluster-1 and change the environment variable CATALOG_HOST to catalog.demo.acme.com.
   
   ![Alt text](/images/shopping_edit.png?raw=true)
   
   11) Create "bookinfo" GNS and include the "bookinfo" namespace on both the clusters. We are using "demo.bookinfo.com" as GNS domain name. 
   
   ![Alt text](/images/bookinfo_gns.png?raw=true)
   
   12) Edit the productpage deployment on bookinfo namespace of Cluster-1 and add environment variable DETAILS_HOSTNAME as below:
   
   ![Alt text](/images/productpage_edit.png?raw=true)
   
   
   
   13) Now cross-cluster communication should be working and bookinfo application is contained on bookinfo namespaces of Cluster-1 and Cluser-2 accessible through the istio-ingressgateway and GNS takes care of cross-cluster communication. 
   
   ![Alt text](/images/bookinfo_gnsview.png?raw=true)
   
   
   <br/>
   <br/>
   
   14) Acme application is contained on default namespaces of Cluster-1 and Cluster-2 and accesible through second-istio-ingressgateway. 
   
   ![Alt text](/images/acme_gns_view.png?raw=true)
   
   
   

  
  
  
  
  
      
  
  
  
  
  
  
  



