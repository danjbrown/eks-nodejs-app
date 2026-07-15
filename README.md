# eks-nodejs-app
Create and deploy a basic Node.js application on AWS EKS.

## Build the application
```
cd app/
docker build -t eks-nodejs-app .

## Very important to build the app on MacOS for linux/amd64 otherwise you will see manifest mismatch errors in the pod logs:
docker build --platform linux/amd64,linux/arm64 -t eks-nodejs-app .
```

## Run the application locally
```
docker run -p 3000:3000 eks-nodejs-app
```

## Create an ECR repository to store the docker image
```
aws ecr-public create-repository --repository-name eks-nodejs-app --region us-east-1
```

This should output the repository URI which needs to be added in /k8s/deploy.yml

```
{
    "repository": {
        ....
        "repositoryUri": "public.ecr.aws/u8p8z3z3/nodejs-kubernetes-app",
        ...
    },
    "catalogData": {}
}
```

## Authenticate the local docker client to the Amazon ECR registry. This gives access to the registry for up to 12 hours
```
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws
```

## Tag the above created image and push to the repository
```
docker tag eks-nodejs-app:latest public.ecr.aws/u8p8z3z3/eks-nodejs-app:latest 
docker push public.ecr.aws/u8p8z3z3/eks-nodejs-app
```

## Create the EKS cluster
```
eksctl create cluster --name eks-nodejs-app --region us-east-1
```

## Configure kubectl to enable it to connect to the EKS cluster by updating the kubectl config (update-kubeconfig)
```
aws eks update-kubeconfig --name eks-nodejs-app --region us-east-1
```

## Apply the Kubernetes deployment config
```
cd k8s/
kubectl apply -f deploy.yml
```

## Apply the Kubernetes load balancer config
```
cd k8s/
kubectl apply -f load-balancer.yml

## Alternatively use the CLI
kubectl expose deployment eks-nodejs-app --port=80 --target-port=3000 --type=LoadBalancer
```

## It may take some minutes for the load balancer to be applied. Find the LoadBalancer EXTERNAL-IP using the below command and load the app in a browser
```
kubectl get services
```

## Update the pods after a new docker image has been pushed with the same tag
```
kubectl rollout restart deployment eks-nodejs-app
```

## Check the Kubernetes pods, service and deployments
```
kubectl get pods
kubectl get services
kubectl get deployments
```

## Check the status of a specific Kubernetes pod (useful for debugging deployment problems)
```
kubectl describe pod <pod_id> | awk '/Events:/,/^$/'
```

## Get the ECR repository image URI used on a specific pod
```
kubectl get pod <pod_id> -o jsonpath='{.spec.containers[*].image}'
```

## Re-deploy after building and pushing a new docker image
```
kubectl delete -f Deployment.yml
kubectl apply -f Deployment.yml
```

## Destroy the app

See https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html

Delete the ECR nodes the AWS CLI (see above URL).

Delete the ECR using the AWS UI.

Delete the EKS cluster
```
eksctl delete cluster --name eks-nodejs-app
```

## Useful links

https://medium.com/geekculture/deploy-a-node-js-cloud-native-application-to-amazon-eks-f6f4c866b742