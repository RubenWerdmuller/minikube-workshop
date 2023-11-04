![k8s](https://kubernetes.io/images/kubernetes-horizontal-color.png)

# What will we be doing?

[Watch the workshop video!](https://www.youtube.com/watch?v=PziYflu8cB8) 🍿🍿

In this workshop, we will walk you through the process of setting up a Kubernetes environment and creating a front-end and API that communicate with each other. We will containerize the API and demonstrate how to access it using Minikube.

## Prerequisites

Before getting started, make sure you have the following prerequisites installed:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [DockerHub account](https://hub.docker.com/signup)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl CLI](https://kubernetes.io/docs/tasks/tools/)
- [k9s interface](https://k9scli.io/)
- Visual Studio Code (VSC)

![k9s](https://cdn-icons-png.flaticon.com/128/194/194279.png)

Additionally, you can use the following command to set the remote origin of your Git repository:

```zsh
git remote set-url origin git@github.com:your-repo.git
```

## Part 1: Setting up Minikube

### Install Minikube

Begin by installing Minikube by following the instructions [here](https://minikube.sigs.k8s.io/docs/start/). After installation, start Minikube with the following command:

```zsh
minikube start
```

> Your kubectl CLI will automatically set the Kubernetes context, indicating the environment you are working with.

### Visualizing with k9s

Instead of relying solely on CLI commands, we will use the k9s interface to monitor our progress. Run k9s using the following command:

```zsh
k9s
```

### Experimenting with Deployments

Let's start by creating a deployment using an Nginx image from Docker Hub:

> In the k9s interface, you can easily switch between different tabs to view various aspects of your Kubernetes environment. To do this, press `Shift` + `:` and then type the name of the tab you want to switch to. In this case we'll be switching to `deployments`

First up is creating a deployment using a [Nginx image](https://hub.docker.com/r/nginxdemos/hello):

> When using the `kubectl CLI` to create a deployment, it's important to note that if you don't specify a Docker registry (i.e., the place where Docker images are hosted), Docker Hub is automatically used as the default registry. This is why we can directly reference the `nginxdemos/hello` image without mentioning a specific registry.

```zsh
kubectl create deployment web --image=nginxdemos/hello
```

Now let's use k9s to debug and see if it's running.

To see if it's running, use k9s:

- Go to the pods tab.
- Port-forward our application using shift-f.
- Set port 80 for the container and a port of your choice for localhost.

Now, open the demo in your [browser](localhost:3000)!

### Exposing the app

Open the service tab using:

```zsh
kubectl expose deployment web --type=NodePort --port=80
```

### Ingress setup

ypically, you would use Ingresses to act as gateways between clients and your Kubernetes cluster. Let's enable and configure Ingress for Minikube:

```zsh
minikube addons enable ingress
minikube addons enable ingress-dns
```

Add your first custom Kubernetes configuration for Ingress:

```sh
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
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

To resolve the URL name, add it to your OS address list:

```zsh
sudo -- sh -c 'echo "127.0.0.1  hello-world.info" >> /etc/hosts'
```

Expose the app to your OS:

```zsh
minikube tunnel
```

You should now be able to access the server through your browser.


## Part 2. Front-end and API

### Creating the API

Create the necessary directories:

```sh
k8s-workshop/
├- front-end/
└─ api/
   └─ index.js
```

Create the API with the following code:

```zsh
npm init
npm i koa
npm i @koa/cors
```

```js
const Koa = require('koa');
const app = new Koa();

app.use(cors());

app.use(ctx => {
  ctx.body = {message: '🐒 Hello world'};
});

app.listen(4000);
```

Add a start script to your package.json:

```zsh
"start": "node index.js"
```

### Settiung up the Front-end

For the front-end, let's use Next.js:

```zsh
npx create-next-app@latest
```

Fetch data from your API by making an HTTP request. Be sure to use a different port for your front-end:

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


## Part 3: Dockerize the API and Deploy to Minikube

Now, let's containerize our API using Docker. Follow the [Docker workshop tutorial](https://github.com/RubenWerdmuller/docker-workshop#dockerizing-our-own-project) to create your Docker files.

Load your Docker image into Minikube:

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

Next, create a YAML file for the API deployment:

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

Apply the deployment to Minikube:

```
kubectl apply -f deployment.yaml
```

Create a service for the API:

```zsh
kubectl expose deployment workshop-api --type=NodePort --port=4000
```

Create a `yaml` file:

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

Apply the Ingress:

```zsh
kubectl apply -f ingress.yaml
```

Test the API with the following command:

```zsh
curl --resolve "workshop-test.info:4000:$(minikube ip)" -i http://workshop-test.info
```

Add the API to your OS hosts and use minikube tunnel to expose it.

Now, you should be able to locally run your front-end and connect it to the Minikube-hosted API. Enjoy your Kubernetes journey!

![heart](https://ih1.redbubble.net/image.1078219052.8316/flat,750x1000,075,f.jpg)
