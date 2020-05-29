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
        kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/platform/kube/bookinfo.yaml -l app!=details -n bookinfo
        kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/platform/kube/bookinfo.yaml -l account=detail -n bookinfo
      ```
3) Deploy gateway and virtualservice for accessing bookinfo through through the default ingressgateway deployed by TSM. Apply the following on cluster-1
      ``` kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.6/samples/bookinfo/networking/bookinfo-gateway.yaml -n bookinfo
   
       ```
4) Lets split acme app and install all the services except catalog in cluster-1 in default namespace and install catalog service in cluster-2 default namespace.

  - Clone Acme app:
       ``` git clone  -b dkalani-dev3    https://github.com/vmwarecloudadvocacy/acme_fitness_demo.git
    
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
        kubectl apply -f acme_fitness_demo/istio-manifests/gateway.yaml
        kubectl apply -f acme_fitness_demo/kubernetes-manifests/acme_fitness_cluster2.yaml
     ```
  
  5) Create "second-ingressgateway" on cluster-1
  
  
  
  
  



