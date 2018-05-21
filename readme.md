## Quick guide to deploying Java apps on OpenShift

Detailed description can be found here: [Quick guide to deploying Java apps on OpenShift](https://piotrminkowski.wordpress.com/2018/05/18/quick-guide-to-deploying-java-apps-on-openshift/)

```
minishift start --vm-driver=virtualbox --memory=3G
eval $(minishift docker-env)
eval $(minishift oc-env)
```
```
git clone https://github.com/marcelomrwin/sample-vertx-kubernetes.git
cd sample-vertx-kubernetes/
git checkout openshift
```
```
mvn clean package -DskipTests
```
```
cd account-vertx-service/
docker build -t marcelomrwin/account-vertx-service .
```
```
cd ../customer-vertx-service/
docker build -t marcelomrwin/customer-vertx-service .
```

### Return to root folder
```
cd ..
oc apply -f openshift/account-deployment.yaml
oc apply -f openshift/customer-deployment.yaml
```

### Create mongodb app
```
oc new-app --template=mongodb-persistent -n myproject \
  MONGODB_USER=mongouser -p \
  MONGODB_PASSWORD=mongopasswd -p \
  MONGODB_DATABASE=mongodb -p \
  MONGODB_ADMIN_PASSWORD=mongopasswd
```

### Update DeploymentConfig with mongodb env vars
```
oc set env --from=secrets/mongodb dc/account-service
oc set env --from=secrets/mongodb dc/customer-service
```

### Create ImageStreams
```
oc apply -f openshift/account-image.yaml
oc apply -f openshift/customer-image.yaml
```

### Deploy images
```
oc login -u developer -p dev
```
### Get token
```
oc whoami -t
```

### Login in docker with token
```
docker login -u developer -p <token> <URL Docker Registry: Ex. 172.30.1.1:5000>
```

### Tag images
```
docker tag <USER>/account-vertx-service <DOCKER_URL>/myproject/account-vertx-service:latest
```
EX: <i>marcelomrwin/account-vertx-service 172.30.1.1:5000/myproject/account-vertx-service:latest</i>
```
docker tag <USER>/customer-vertx-service <DOCKER_URL>/myproject/customer-vertx-service:latest
```
EX: <i>docker tag marcelomrwin/customer-vertx-service 172.30.1.1:5000/myproject/customer-vertx-service:latest</i>

### Push images
```
docker push 172.30.1.1:5000/myproject/account-vertx-service:latest
docker push 172.30.1.1:5000/myproject/customer-vertx-service:latest
```

### Create services
```
oc apply -f openshift/account-service.yaml
oc apply -f openshift/customer-service.yaml
```

### Create routes
```
oc expose svc/account-service
oc expose svc/customer-service
```

### Running deployments
```
oc rollout latest dc/account-service
oc rollout latest dc/customer-service
```
