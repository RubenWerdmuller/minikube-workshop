![k8s](https://kubernetes.io/images/kubernetes-horizontal-color.png)

# What will we be doing?

[Watch a video!](https://www.youtube.com/watch?v=PziYflu8cB8) 🍿🍿

1. We'll kickstart Minikube and test it out a bit
2. We'll be creating a front-end and API that can talk to each other 🗣⋆.ೃ࿔*
3. We'll containerize the API and see if we can reach the API using Minikube
<!-- 4. And last but not least, we'll attempt giving the front-end up in Minikube -->

## Prerequisites

To run everything we'll want:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [DockerHub account](https://hub.docker.com/signup)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl CLI](https://kubernetes.io/docs/tasks/tools/)
- VSC

Useful snippet:

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

### Expose the app

Open the service tab

```zsh
kubectl expose deployment web --type=NodePort --port=80
```

### Ingress

<!-- Volg de ingress setup voor minikube
https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/
-->

Alright! Now generally you'll want to use Ingresses (loadbalancers/proxies) to act as gateway between a client and our Kubernetes. Let's spin a sucker up.

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

Add it to your OS hosts list, so we can see it in the browser:

```zsh
sudo -- sh -c 'echo "127.0.0.1  hello-world.info" >> /etc/hosts'
```

HODOR for our OS! 🚪

```zsh
minikube tunnel # opens up all ingresses to our OS
```

Now you should be able to reach the server through your browser!


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

Oh, and finally, add something to run the whole!

```zsh
"start": "node index.js"
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
        const body = await response.json();
        console.log(body);
      }
    })();
  }, []);
```


## 3. een eigen Docker image maken en draaien

We're going to containerize our API 🐵

Since there is a separate tutorial about using Docker, [let's visit that one](https://github.com/RubenWerdmuller/docker-workshop#dockerizing-our-own-project) to create our Docker files!

### de applicatie in Minikube

```
minikube image load my-image
```

<!--
```zsh
# Set docker env
eval $(minikube docker-env)             # Unix shells
minikube docker-env | Invoke-Expression # PowerShell

# Build image
docker build -t foo:0.0.1 .
```
-->

This time, we're going to create a `yaml` file for our deployment 🥂

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: workshop-api
  labels:
    app: web
spec:
  selector:
    matchLabels:
      app: web
  replicas: 1
  strategy:
    type: RollingUpdate
    template:
      metadata:
        labels:
          app: web
    spec:
      containers:
        - name: workshop-api
          image: workshop-api
          imagePullPolicy: Never
          ports:
            - containerPort: 4000
```

Now apply it to our mini 🚗

```
kubectl apply -f deployment.yaml
```

The next steps are quite similar as before 🪜

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: workshop-api
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
    - host: workshop-api.info
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: workshop-api
                port:
                  number: 4000
```

Create a service

```zsh
kubectl expose deployment workshop-api --type=NodePort --port=3000
```

Test it with this command I plugged without looking from the k8s docs

```zsh
url --resolve "workshop-test.info:4000:$( minikube ip )" -i http://workshop-test.info
```

And finally add it to your OS host and tunnel away.

The next step would be to locally run your front-end and see if you can get it to connect to your minikube hosted API.

Cheers!

![heart]([https://cdn4.vectorstock.com/i/1000x1000/20/38/hand-making-small-heart-sign-vector-28932038.jpg](https://upload.wikimedia.org/wikipedia/commons/e/e6/Finger_heart.png)https://upload.wikimedia.org/wikipedia/commons/e/e6/Finger_heart.png)
