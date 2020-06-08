# MyKafkaServiceRepo
Welcome to Kafka Service deployment on Kubernetes using Kustomize !!!!!

follow below steps for starting a Kafka service on a Kubernetes cluster using Kustomize


Steps:

1. Make sure you clone the repo locally.

git clone https://github.com/opchilling/MyKafkaServiceRepo.git

2. once cloned, navigate to repo folder
cd ~/MyKafkaServiceRepo/MyKafkaService/

3.  Make sure to have the latest kubectl installed because Kustomize is already included (kubectl kustomize). Otherwise install it manually.

4. We will be starting the zookeeper service first and deploying it before starting the Kafka message broker service on our cluster.

5. Next we run “kubectl kustomize build base”. This outputs a multi resource file of both Service and Deployment

6.  For a customized Dev Environment Run “kubectl kustomize build overlays/dev” to use all the base resources and merge with the deployment.yaml for test environment

7.  For the Production environment, we customized to use a stable image, increase replica count and also the service type.
kubectl kustomize build overlays/prod | kubectl apply -f -
