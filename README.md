# splunk-k8s

## Deploy to Kubernetes
```
kubectl create -f deployment.yml 
kubectl create -f service.yml

kubectl get pods -l app=splunk
kubectl port-forward <splunk pod name> 8000:8000 
```

Open a browser and navigate to localhost:8000
login as `admin/simple123`

Now we can enable and configure HTTP Event Collector.
In the configuration, we will specify the port (e.g. 8088) of the HTTP Event Collector.

Once that is done, any app running in the Kubernetes cluster can send logs
to the ClusterIP:8088 of the Splunk service. 

Finally after we are done, we can remove the service and pod:
```
kubectl delete -f deployment.yml
kubectl delete -f service.yml
```

## Run Splunk Enterprise inside a Docker container

https://docs.splunk.com/Documentation/Splunk/7.2.4/Installation/DeployandrunSplunkEnterpriseinsideDockercontainers

Splunk 7.2.4 container gives me the error reported in this issue:
https://github.com/splunk/docker-splunk/issues/96

So I use Splunk 7.2.1 instead.

```
docker pull splunk/splunk:7.2.1

docker run -p 8000:8000 -p 8088:8088 -e 'SPLUNK_START_ARGS=--accept-license' -e 'SPLUNK_PASSWORD=simple123' splunk/splunk:7.2.1
```

Open a browser and navigate to localhost:8000
login as `admin/simple123`

## Configure HTTP Event Collector
https://docs.splunk.com/Documentation/Splunk/7.2.1/Data/UsetheHTTPEventCollector

First we need to enable HTTP Event Collector:

Click Settings > Data Inputs.
Click HTTP Event Collector.
Click Global Settings.

## Send data to Splunk
Assume the HTTP Event Collector listens on port 8088.
We can send logs to the HTTP Event Collector by running the following commands.
You need to replace <token> in the commands with real tokens of your setup.
The tokens should look like ccf28cf1-xxxx-xxxx-xxxx-eb9f9b7d06cf.

```
curl -k  https://localhost:8088/services/collector/event -H "Authorization: Splunk <token>" -d '{"event": "hello world"}'

curl -k  https://localhost:8088/services/collector/event 
-H "Authorization: Splunk <token>" 
-d '{"time": 1551133031, "host": "localhost", "source": "datasource", "index": "main", "event": "{\"message\": \"Hello world7\"}"}'

curl -k  https://localhost:8088/services/collector/event 
-H "Authorization: Splunk <token>" 
-H "Content-Type: application/json" 
-d '{"sourcetype": "json", "time": 1551165205, "host": "localhost", "source": "datasource", "index": "main", "event": "{\"message\": \"Hello world7\"}"}'
```

## data not ingested into Splunk due to lack of disk space
Sending data to Splunk returns 200 ok. However, data is dropped and not indexed. 
It turns out that the Splunk docker container is low on disk space. 
Once I free up some disk space by deleting some docker images, the whole app runs much faster 
and I am able to send and query the data.

https://answers.splunk.com/answers/509387/how-to-view-events-from-http-event-connector-1.html

https://answers.splunk.com/answers/686236/the-http-event-collector-hec-accepts-but-doesnt-in.html

https://answers.splunk.com/answers/670994/events-posted-to-hec-successfully-not-showing-up-i.html

