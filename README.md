# DevOps-deployment
Deployment scripts to recreate a microservices application to recommend playlist

```Deployment Scripts for running project ```

The deployment scripts would create 2 worker nodes within EKS cluster.
All routing is managed internally by Service Names, so no IP's needs to be edited(exception Admin app, see further details)

## Initial set up
Keep you Docker up and running
Clone the project, and step into the infra_deployment
```bash
terraform init
terraform plan
terraform apply
```
This will give an output for config file and config-map-aws-auth

Replace local C://User/sura/.kube/config file with this config

Copy the config-map-aws-auth into a YAML file and run the following command

``` bash
kubectl apply -f config-map-aws-auth.yaml 
```

## Running the User story setup within EKS Cluster
Step into the deployments_services folder

Run the following YAML files in "SEQUENCE" (this is important since mysql needs to be in place before the persistent volume is created)
This will create Load balancers for all apps required by external User
```
kubectl apply -f mysql-deployment.yaml
kubectl apply -f mysql-pv.yaml
kubectl apply -f mongo.yml
kubectl apply -f billboard.yml
kubectl apply -f sentiment.yml
kubectl apply -f spring-boot.yml
kubectl apply -f react-app.yml
kubectl get deployments
kubectl get svc 
```

## Intermediate set up for Admin
Once the springboot external IP is generated, we need to follow the below steps

An improtant part of the project is to hit the 'springboot' external API with /initPlaylist [POST] for initial playlists to load

Step into local Admin app, and follow the below steps to edit JS file with springboot API and Build/Containerize app
This is the only manual step where our JavaScript file could not resolve service names
```bash
delete out and publish folders
cd D:\Courseware\MIS at NEU\<>\dotnetAdmin\adminApp\wwwroot\js
EDIT the file, with spring-boot IP as generated check 'kubectl get svc'
dotnet build
dotnet publish -o out
cd ..
docker build -t <dockername/repo> .
docker push <dockername/repo>
cd into deployments_services run:
kubectl apply -f dotnet-app.yml 
```

## Final set up for Grapfana and Promethues
Step into monitoring
cd D:\Courseware\MIS at NEU\DevOps\<>\DeploymentEKS\services_deployment\monitoring

Run the following commands
```bash
kubectl create namespace monitoring
kubectl create -f clusterRole.yaml -n monitoring
kubectl create -f config-map.yaml -n monitoring
kubectl create -f prometheus-deployment.yaml -n monitoring
kubectl create -f prometheus-service.yaml -n monitoring
kubectl create -f grafana-datasource-config.yaml -n monitoring
kubectl create -f grafana-datasource-deploy.yaml -n monitoring
kubectl create -f grafana-datasource-service.yaml -n monitoring
kubectl get deployments -n monitoring
kubectl get svc -n monitoring
kubectl get svc -n monitoring
```

## Accessing the IP's

You need to wait approx 2mins before each deployment is up and running, you and keep a check on the system status using below commands:
```
kubectl get deployments
kubectl get pods
kubectl describe pod <podname> => to see error
kubectl get deployment -n monitoring
kubectl get svc -n monitoring
kubectl logs -f <podname>
kubectl exec -it <mysql-podname> /bin/bash
mysql -u <rootuser as in springboot> -p
kubectl get pv
kubectl get persistentvolumeclaim 
```

## Reclaiming resources
Once you are done, run the below commands in no particular order, except terraform destroy to be last
```
kubectl delete deployment mysqldeploy
kubectl delete deployment mongodeploy
kubectl delete deployment springdeploy
kubectl delete deployment billboarddeploy
kubectl delete deployment sentimentdeploy
kubectl delete deployment reactdeploy
kubectl delete deployment aspdeploy
kubectl delete deployment grafana -n monitoring
kubectl delete deployment prometheus-deployment -n monitoring

kubectl delete svc mysqldevops
kubectl delete svc mongodevops
kubectl delete svc springdevops
kubectl delete svc billboarddevops
kubectl delete svc sentimentdevops
kubectl delete svc reactdevops
kubectl delete svc aspdevops
kubectl delete svc grafana -n monitoring
kubectl delete svc prometheus-service -n monitoring
terraform destroy
```
