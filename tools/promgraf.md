
# Prometheus Setup

run following commands one by one 

```
minikube start

kubectl get pods -A
```
helm installation on debian/ubuntu
```
sudo apt-get install curl gpg apt-transport-https --yes
curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
install prometheus using below given two commands
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update
```
to see all the pods and services run below given commands
```
kubectl get pods

kubectl get svc
```

expose the prometheus server
```
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext

kubectl get svc 
```

after last command prometheus-server-ext would be visible amoung services and note down the port corresponding to service "prometheus-server-ext"

```
minikube ip
```

copy the ip address and past below url in browser of debian with replacing ip with copied "ip" and "port" with port of prometheus-server-ext 

```
http://ip:port 
```
you can observe your prometheus website in front of you 


# Grafana setup

```
helm repo add grafana https://grafana.github.io/helm-charts

helm repo list

helm repo update

helm install my-grafana grafana/grafana
```

copy the below command to get password of grafana 

```
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

copy the obtained password 

```
kubectl get svc
```

expose grafana service 
```
kubectl expose service grafana --type=NodePort --target-port=3000 --names=grafana-ext 

kubectl get svc 
```

note the port of grafana-ext 

```
minikube ip 
```
notedown the obtained ip address 

enter the url again in browser with new ip and port 

login using the grafana password 

after login click on "add data source"

click on prometheus 

enter url of prometheus with both ip and port as we have entered earlier 

go to "dashboards" option now 

click on "import" 

enter "3663" in ID 

choose default option prometheus and click on import 

you can observer dashboard in front of you 



