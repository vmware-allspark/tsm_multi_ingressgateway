# tsm_multi_ingressgateway
Manifest to deploy additional ingress gateways in Tanzu Service Mesh managed cluster. 

1) Onboard a client K8s cluster on Tanzu Service Mesh (TSM). 
2) Once onboarded TSM will install and manage the lifecycle of Istio on the onboarded Cluster. 
3) By default TSM creates one istio ingressgateway deployment per cluster, however for some usecases Customers might want multiple istio ingressgateway. 
4) For scenarios where second ingressgateways are needed, peform "kubectl apply -f ingressgateway-manifest/second-ingressgateway.yaml" to deploy a second ingressgateway. 
5) For more than two ingressgateways, replace "second" in the yaml with different prefix to add more ingressgateways. 




