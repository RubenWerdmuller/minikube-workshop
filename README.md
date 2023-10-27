# minikube-workshop

https://github.com/neefrehman/manyworlds

```zsh
git remote set-url origin git@github.com:RubenWerdmuller/minikube-workshop.git
```

```zsh
minikube start
```

https://k9scli.io/

![k9s](https://cdn-icons-png.flaticon.com/128/194/194279.png)

```zsh
k9s
```

We gaan een server met front-end installeren op een proxy/loadbalancer server

Nginx

Naar tab `deployments`

```
kubectl create deployment web --image=nginxdemos/hello
```

Naar tab `services`

```
kubectl expose deployment web --type=NodePort --port=80
```

### testen 

draait die

```
kubectl get service web
```

Using minikube

```sh
minikube service web --url
```

### ingress

Volg de ingress setup voor minikube
https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/

```zsh
minikube addons enable ingress
minikube addons enable ingress-dns
```

we maken een eigen 

```sh
cat <<EOF | kubectl apply -f 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx" # required for ios
spec:
  rules:
    - host: hello-world.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
EOF
```

```zsh
minkube tunnel # opent alle ingresses voor ons
```

### een eigen Docker image maken en draaien

add docker
https://github.com/RubenWerdmuller/docker-workshop

```zsh
# Set docker env
eval $(minikube docker-env)             # Unix shells
minikube docker-env | Invoke-Expression # PowerShell

# Build image
docker build -t foo:0.0.1 .
```

```
# Run in Minikube
kubectl run hello-foo --image=foo:0.0.1 --image-pull-policy=Never
```
