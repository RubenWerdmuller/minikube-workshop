Presentatie:

- fireship k8s https://www.youtube.com/watch?v=PziYflu8cB8
- fireship docker https://www.youtube.com/watch?v=Gjnup-PuquQ
- kort resume over de presentatie
  - CaC Config as Code
    - yaml bestanden
  - k8s is self healing
- Kubernetes is VET moeilijk, maar hey. Ik heb een crash-crash-crash-course bedacht die je er mee kan laten werken, zonder dat je ALLES hoeft te begrijpen en je kan hiermee voortaan kleine stukjes kennis gaan toevoegen.

![k8s](https://kubernetes.io/images/kubernetes-horizontal-color.png)

# What will we be doing?

1. We'll kickstart Minikube and test it out a bit
2. We'll be creating a front-end and API that can talk to each other 🗣⋆.ೃ࿔*
3. We'll containerize the API and see if we can reach the API using Minikube
4. And last but not least, we'll attempt giving the front-end up in Minikube

## Prerequisites

To run everything we'll want:

- Docker Desktop
- An account with DockerHub
- Minikube
- kubectl CLI
- VSC

Useful snippets:

```zsh
git remote set-url origin git@github.com:your-repo.git
```

## 1. Minikube

Install [Minikube](https://minikube.sigs.k8s.io/docs/start/)

```zsh
minikube start
```

> your kubectl CLI can be used to indicate what kubernetes context (read: environment) you are working with. After starting `minikube`, it will set the context for you automatically.

### Seeing everything in action

Rather than getting too hung up on CLI commands, we'll use the [k9s interface](https://k9scli.io/) to check in on our progress.

![k9s](https://cdn-icons-png.flaticon.com/128/194/194279.png)

```zsh
k9s
```

### Getting a feel for it

Let's do a small experiment and run a webserver with front-end page.

> To switch tabs in k9s, use `shift :` and type out your tab, for example, `pods`.

Go to k9s tab `deployments`.

First up is creating a deployment using a [Nginx image](https://hub.docker.com/r/nginxdemos/hello):

> If no Docker registry (read: place where you get your Docker images) from is mentioned in the kubectl CLI command, Docker Hub is automatically used as registry. That's why we can instantly call `nginxdemos/hello` as image

```
kubectl create deployment web --image=nginxdemos/hello
```

Now let's use k9s to debug and see if it's running.

- Go to the tab `pods`
- Port-forward our application using `shift-f`
- The Nginx demo is using port 80, but we can't forward that. We can however set port 80 for the container port and something we want to use for our `localhost`.

Open the demo up in your [browser](localhost:3000)!

### Ingress

<!-- Volg de ingress setup voor minikube
https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/
-->

Alright! Now generally you'll want to use Ingresses (loadbalancers/proxies) to act as gateway between a client and our Kubernetes. Let's spin these suckers up.

```zsh
minikube addons enable ingress
minikube addons enable ingress-dns
```

We'll add our first custom kubernetes configuration!

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
minikube tunnel # opens up all ingresses to our OS
```


## 2. Front-end and API


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
  ctx.body = {message: '🐒 Hello world'};
});

app.listen(4000);
```

### The front-end

As `Next.js` is still our favorite!

```zsh
npx create-next-app@latest
```

Now add a fetch to our API somewhere so we know we're in contact!

```js
  useEffect(() => {
    (async () => {
      const response = await fetch("localhost:4000/");

      if (response.ok) {
        const configuredCredentials = await response.json();
        setUserHas2faConfigured(!!configuredCredentials.totp);
      }
    })();
  }, []);
```

<!-- https://github.com/neefrehman/manyworlds -->


### 3. een eigen Docker image maken en draaien

Since there is a separate tutorial about using Docker, [let's visit that one](https://github.com/RubenWerdmuller/docker-workshop#dockerizing-our-own-project) to create our Docker files!


https://betterstack.com/community/questions/how-to-use-local-docker-images-with-minikube/

```
minikube image load my-image
```

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
