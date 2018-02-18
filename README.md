kubernetes-multicontainer-pod Node.js on GCP Kubernetes
===============================================================

You can run these steps in Kubernetes GCP free-tier.
[TODO Purpose]
[TODO Image]

### Step 0: Clone this repo
```
git clone https://github.com/dzuluaga/kubernetes-multicontainer-pod-lab.git
```

### Step 1: Test nodejs-express-api locally
Open one terminal window to start the server.
```bash
cd nodejs-express-api
node server.js
curl http://localhost:8080/hello -H 'Skip-Backend: true'
```
Response:
```
Hello world-API
```
Note `Skip-Backend` header will skip call to the backend.

### Step 2: Test nodejs-express-backend locally
Open another terminal window to start the server. And execute the following commands:

```bash
cd nodejs-express-backend
node server.js
curl http://localhost:8081
```
Response:
```
Hello world-backend 
```

### Step 3: Test nodejs-api calling nodejs-backend

Execute
```bash
curl http://localhost:8080/hello
```
Response:
```
Hello world-backend
```
**Note that response includes `-backend` word in it. Therefore, nodejs-api is serving as api proxy to nodejs-express-backend.**

In the next steps we will deploy a multicontainer pod with these two apps.

### Step 4: Login to console.cloud.google.com start CloudShell
Create a cluster of 3 VMs f1-micro for our two node.js apps.

```
gcloud container clusters create multi-container-pod --num-nodes=3 --machine-type=f1-micro --zone=us-west1-a
```

### Step 5: Set PROJECT_ID environment variable
```
export PROJECT_ID="$(gcloud config get-value project -q)"
```

### Step 6: Build a Docker image of nodejs-express-api
```bash
cd nodejs-express-api
docker build -t gcr.io/${PROJECT_ID}/nodejs-express-api:v1 .
```

Push Docker build to Google Container Registry:
```
gcloud docker -- push gcr.io/${PROJECT_ID}/nodejs-express-api:v1
```

### Step 7: Build a Docker image of nodejs-express-backend

```
cd ../nodejs-express-backend
docker build -t gcr.io/${PROJECT_ID}/nodejs-express-backend:v1 .
```

Push Docker build to Google Container Registry:
```
gcloud docker -- push gcr.io/${PROJECT_ID}/nodejs-express-backend:v1
```

### Step 8: Create our pod
Edit `Kubernetes.yaml` and replace `$GCP_PROJECT` token with your GCP project.
```bash
$ kubectl apply -f Kubernetes.yaml
```

### Step 9: Expose port

```bash
$ kubectl expose pod kubernetes-multi-container-pod --type=LoadBalancer --port=80 --target-port=8080
```

**Get Public IP Address**
```
$ kubectl get service
```
Response:
```
NAME                             TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
kubernetes-multi-container-pod   LoadBalancer   10.35.249.82   35.230.95.143   80:32720/TCP   1m
```

### Step 10: Test access from nodejs-api to nodejs-backend

