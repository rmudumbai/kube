# Kube

#Google list of commands to check


gcloud auth list
gcloud config list project
gcloud config set compute/zone us-central1-a


# Get sample code

git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git
cd orchestrate-with-kubernetes

#Create a cluster with five n1-standard-1 nodes (this will take a few minutes to complete):

gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"



kubectl explain deployment
kubectl explain deployment --recursive
kubectl explain deployment.metadata.name

#Create deployment
vi deployments/auth.yaml


# Create deployment
kubectl create -f deployments/auth.yaml


kubectl get deployments
kubectl get replicasets
kubectl get pods


# Create Service
kubectl create -f services/auth.yaml
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml


#Create fronend  deployment
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml


curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`



#SCALE scale deployment

kubectl explain deployment.spec.replicas
kubectl scale deployment hello --replicas=5

#verify scale
kubectl get pods | grep hello- | wc -l

#scale back
kubectl scale deployment hello --replicas=3
#verify
kubectl get pods | grep hello | wc -l


#rolling update
kubectl edit deployment hello 
#change the version by editing image name

kubectl get replicaset
kubectl rollout history deployment/hello

#pause the deployment
kubectl rollout pause deployment/hello
kubectl rollout status deployment/hello

#verify pods directly
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'


kubectl rollout resume deployment/hello
kubectl rollout status deployment/hello


#rollback an update
kubectl rollout undo deployment/hello

#verify rollback rollout
kubectl rollout history deployment/hello

#verify pods have rolled back
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

#create canary
kubectl create -f deployments/hello-canary.yaml

kubectl get deployments
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

# change the session affinity in the 
spec:
  sessionAffinity: ClientIP 

# blue green deployments
kubectl apply -f services/hello-blue.yaml



# Updating using Blue-Green Deployment
# green deployment update version label and the image path

kubectl create -f deployments/hello-green.yaml

curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version

#update the service to point to green deployment
kubectl apply -f services/hello-green.yaml

#rollback to the blue deployment
kubectl apply -f services/hello-blue.yaml


# verify the version
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
