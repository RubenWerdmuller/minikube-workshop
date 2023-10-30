# minikube-workshop

What will we be doing?

1. We'll be creating a front-end and API that can talk to each other 🗣⋆.ೃ࿔*
2. We'll kickstart Minikube and test it out a bit
3. We'll containerize the API and see if we can reach the API using Minikube
4. And last but not least, we'll attempt giving the front-end up in Minikube

## 1. Prerequisites

We'll start out by creating the applications and create working `Docker images` for these.

To run everything we'll want:

- Docker Desktop
- Minikube
- kubectl CLI
- VSC

Useful snippets:

```zsh
git remote set-url origin git@github.com:your-repo.git
```

### The API

Create your directories.

```sh
k8s-workshop/
├- front-end/
└─ api/
   └─ index.js
```

```zsh
npm init
npm i koa
```

```js
const Koa = require('koa');
const app = new Koa();

app.use(ctx => {
  ctx.body = 'Hello Koa';
});

app.listen(4000);
```

### The front-end

As Next.js is still our favorite.

```zsh
npx create-next-app@latest
```

Now add a fetch to our API somewhere so we know we're in contact!

```js
await fetch("localhost:4000")
```

<!-- https://github.com/neefrehman/manyworlds -->


## 2. Minikube

Install [Minikube](https://minikube.sigs.k8s.io/docs/start/)

```zsh
minikube start
```


### Seeing everything in action

Rather than getting too hung up on CLI commands, we'll use the [k9s interface](https://k9scli.io/) to check in on our progress.

![k9s](https://cdn-icons-png.flaticon.com/128/194/194279.png)

```zsh
k9s
```

### Getting a feel for it

Let's do a small experiment and run a webserver with front-end page.

To switch tabs, use `shift :`.

**Nginx**

Go to k9s tab `deployments`.

First up is creating a deployment:

```
kubectl create deployment web --image=nginxdemos/hello
```

Go to k9s tab `services`.

Next we'll use a debug feature called `expose` to see if we can access our deployment.

```
kubectl expose deployment web --type=NodePort --port=80
```

# wil ik echt expose gebruiken en waarom werkt port-forward van k9s niet?

### testen 

draait die

```
kubectl get service web
```

Using Minikube

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

### 3. een eigen Docker image maken en draaien

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
