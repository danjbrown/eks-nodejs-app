# eks-nodejs-app
EKS deployed Node.js application

## Very important to build the app on MacOS for linux/amd64
```
docker build --platform linux/amd64,linux/arm64 -t eks-nodejs-app .
```

## Run the app locally
```
docker run -p 3000:3000 demo/eks-nodejs-app
```

## Create a new ECR repository to store the docker image
```
aws ecr-public create-repository --repository-name eks-nodejs-app --region us-east-1
```

## Authenticate our local machine docker client to the Amazon ECR registry. This gives access to the registry for up to 12 hours
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

## Configure kubectl to enable it to connect to the EKS cluster by updating the kubectl config (update-kubeconfig) with the cluster endpoint
```
aws eks update-kubeconfig --name eks-nodejs-app --region us-east-1
```

## Apply the K8 config
```
kubectl apply -f Deployment.yml
```

## Check K8 pods
```
kubectl get pods
kubectl get servcices
kubectl get deployments
```

## Create a Service with the type of Load Balancer to expose the application port to the outside world
```
kubectl expose deployment eks-nodejs-app --port=80 --target-port=3000 --type=LoadBalancer
```

## After bduiling and pushing a new docker image
```
kubectl delete -f Deployment.yml
kubectl apply -f Deployment.yml
```

## Destroy the app

See https://docs.aws.amazon.com/eks/latest/userguide/delete-cluster.html

Delete the ECR using the AWS UI.

Delete the EKS cluster
```
eksctl delete cluster --name eks-nodejs-app
```

## Reference

https://medium.com/geekculture/deploy-a-node-js-cloud-native-application-to-amazon-eks-f6f4c866b742