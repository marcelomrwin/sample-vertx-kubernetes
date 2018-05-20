## Quick guide to deploying Java apps on OpenShift

Detailed description can be found here: [Quick guide to deploying Java apps on OpenShift](https://piotrminkowski.wordpress.com/2018/05/18/quick-guide-to-deploying-java-apps-on-openshift/)

# A Quick Guide to Deploying Java Apps on OpenShift

```
minishift start --vm-driver=virtualbox --memory=3G
eval $(minishift docker-env)
eval $(minishift oc-env)
```
```
git clone git@github.com:marcelomrwin/sample-vertx-kubernetes.git
cd sample-vertx-kubernetes/
git checkout openshift
```

mvn clean package -DskipTests

cd account-vertx-service/
docker build -t marcelomrwin/account-vertx-service .

cd ../customer-vertx-service/
docker build -t marcelomrwin/customer-vertx-service .

# return to root folder
cd ..
oc apply -f openshift/account-deployment.yaml
oc apply -f openshift/customer-deployment.yaml

#create mongodb app

oc set env --from=secrets/mongodb dc/account-service
oc set env --from=secrets/mongodb dc/customer-service

oc apply -f openshift/account-image.yaml
oc apply -f openshift/customer-image.yaml

#deploy images
oc login -u developer -p dev
#get token
oc whoami -t

#login in docker with token
docker login -u developer -p <token> <URL Docker Registry: Ex. 172.30.1.1:5000>

#Tag images
docker tag marcelomrwin/account-vertx-service 172.30.1.1:5000/myproject/account-vertx-service:latest
docker tag marcelomrwin/customer-vertx-service 172.30.1.1:5000/myproject/customer-vertx-service:latest

#Push images
docker push 172.30.1.1:5000/myproject/account-vertx-service:latest
docker push 172.30.1.1:5000/myproject/customer-vertx-service:latest

#create services
oc apply -f openshift/account-service.yaml
oc apply -f openshift/customer-service.yaml

#create routes
oc expose svc/account-service
oc expose svc/customer-service

#Running deployments
oc rollout latest dc/account-service
oc rollout latest dc/customer-service
