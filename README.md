
# Running MQ with KEDA. 

## Contents
1.  [Install MQ on Cloud](#install-mq-on-cloud)
1.  [Setting up the Sample App](#setting-up-the-sample-app)
1.  [Optional: Configure message numbers and sleep time](#optional-configure-message-numbers-and-sleep-time)
1.  [Installing KEDA](#installing-keda)
1.  [Running the Demo](#running-the-demo)

## Install MQ on Cloud
1. [Sign Up](https://cloud.ibm.com/registration) for an IBM Cloud Account.
1. Create an [MQ on Cloud](https://cloud.ibm.com/catalog/services/mq?cm_sp=ibmdev-_-developer-tutorials-_-cloudreg) instance on your account.
1. Follow [this guide](https://developer.ibm.com/tutorials/mq-connect-app-queue-manager-cloud/) to:
	1. Create a Queue Manager
	1. Register an application
	1. Generate API and connection credentials

## Setting up the Sample App
1. Clone the Sample App Repo
1. Create a `secrets.yaml` file inside the `deploy` folder using the following struct:
	```yaml
	apiVersion: v1
	kind: Secret
	metadata:
	  name: ibmmq-secret
	type: Opaque
	data:
	  APP_USER: # Your MQ app username
	  APP_PASSWORD: # Your MQ app password
	  ADMIN_USER: # Your MQ admin username
	  ADMIN_PASSWORD: # Your MQ admin password
	  ```
1. Replace the `APP_USER` `APP_PASSWORD` `ADMIN_USER` and `ADMIN_PASSWORD` fields with the corresponding credentials from your MQ application. The values need to be encoded in a Base64 format.
1. In the `deploy-publisher.yaml` and `deploy-consumer.yaml` files, update the environment variables by supplying the `QMGR` `QUEUE_NAME` `HOST` `PORT` `CHANNEL` and `TOPIC_NAME` with the corresponding credentials from your MQ application.

## Optional: Configure message numbers and sleep time
- You can change the number of messages to send by editing the `args` field in the `deploy-publisher.yaml` file.
- You can change the time each consumer sleeps before reading in the next message, this is useful for demonstrating scaling. To change the sleep time (in seconds) edit the `args` field in the `deploy-consumer.yaml` file.

## Installing KEDA
1. Install Keda:  
	```
	kubectl apply -f https://github.com/kedacore/keda/releases/download/v2.0.0-beta/keda-2.0.0-beta.yaml
	```

## Running the Demo
1. Apply the secret YAML file:  
	```
	kubectl apply -f deploy/secrets.yaml
	```
1. Run the consumer:  
	```
	kubectl apply -f deploy/deploy-consumer.yaml
	```
	The application should automatically scale to 0, as there are no mesages no the queue. You can confirm this by running `kubectl get hpa` where the number of `replicas` should be 0.
	
1. Run the producer:  
	```
	kubectl apply -f deploy/deploy-publisher.yaml
	```
1. Watch as KEDA scales the consumers to handle the message load:
	```
	kubectl get hpa -w
	```
	The consumers should scale up to handle the message load, then return to 0 once all of the messages have been stripped from the queue:
	```
	NAME                      REFERENCE                   TARGETS             MINPODS   MAXPODS   REPLICAS   AGE
	keda-hpa-ibmmq-consumer   Deployment/ibmmq-consumer   <unknown>/5 (avg)   1         30        0          0s
	keda-hpa-ibmmq-consumer   Deployment/ibmmq-consumer   64/5 (avg)          1         30        1          61s
	keda-hpa-ibmmq-consumer   Deployment/ibmmq-consumer   4250m/5 (avg)       1         30        4          2m1s
	keda-hpa-ibmmq-consumer   Deployment/ibmmq-consumer   <unknown>/5 (avg)   1         30        0          3m2s
	```
