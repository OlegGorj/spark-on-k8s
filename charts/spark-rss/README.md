# A Helm chart for Spark Resource Staging Server
This chart is still work-in-progress

## Chart Details
This chart launches [Spark Resource Staging Server](https://apache-spark-on-k8s.github.io/userdocs/running-on-kubernetes.html#dependency-management).
Spark Resource Staging Server is used for dependency management when Spark is run in Kubernetes environment.
 
## Installing the Chart

1. By default this chart deploys Spark Resource Staging Server and configures a service of type 
LoadBalancer to access it.

    For example:

    ```
    helm install --name rss ./spark-rss/
    ```
    
    The above command will display output such as given below:
    ```
    NOTES:
    Get the resource staging server URI by running these commands:
         NOTE: It may take a few minutes for the LoadBalancer IP to be available.
               You can watch the status of by running 'kubectl get svc -w rss-spark-rss'
      export SERVICE_IP=$(kubectl get svc --namespace default rss-spark-rss -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
      echo http://$SERVICE_IP:10000
    ```
    
    The URI displayed by using the commands given in the above output can be used to run 
    spark-submit command with resource staging server. 
    
2. Users also have a choice of using NodePort service instead of LoadBalancer. To use a  
   NodePort service use a command such as given below or alternatively modify 
   values.yaml and specify the service.type value as "NodePort"
     
     ```
     helm install --name rss --set service.type=NodePort ./spark-rss/
     ```
     
     The above command will display output such as given below:
     
     ```
     NOTES:
     Get the resource staging server URI by running these commands:
       export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services rss-spark-rss)
       export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[1].address}")
       echo http://$NODE_IP:$NODE_PORT
     ```
    NOTE: To access NodePort service externally, create a firewall rule that allows TCP traffic on your node port.
    For example, if Service has a NodePort value of 31000, create a firewall rule that allows TCP traffic on port 31000.
    Different cloud providers offer different ways of configuring firewall rules. Without the firewall you may not be
    able to use resource staging server as spark-submit will throw error.    
 
####spark-submit command example
Given below is an example spark-submit command that uses resource staging server. 

    
    bin/spark-submit \
      --deploy-mode cluster \
      --class org.apache.spark.examples.SparkPi \
      --master k8s://<k8s-apiserver-host>:<k8s-apiserver-port> \
      --kubernetes-namespace default \
      --conf spark.executor.instances=5 \
      --conf spark.app.name=spark-pi \
      --conf spark.kubernetes.driver.docker.image=shirishd/spark-driver:v2.2.0-kubernetes-0.5.0 \
      --conf spark.kubernetes.executor.docker.image=shirishd/spark-executor:v2.2.0-kubernetes-0.5.0 \
      --conf spark.kubernetes.initcontainer.docker.image=shirishd/spark-init:v2.2.0-kubernetes-0.5.0 \
      --conf spark.kubernetes.resourceStagingServer.uri=<URI of resource staging server as displayed on console while deploying it> \
      ./examples/jars/spark-examples_2.11-2.2.0-k8s-0.5.0.jar
    
    
     
## Deleting the chart
Use `helm delete` command to delete the chart
   ```
   $ helm delete --purge rss
   ```