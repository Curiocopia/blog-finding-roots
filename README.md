# 📦 Finding Roots

This is a visualization of the Newton-Raphson root finding method. Details are described in [Finding Roots].

## 🌟 Highlights

- Uses Redis work queue for excahnging movie files generated for iterations of Newton-Raphson root finder 
- Communicates (via requests) with a backend-service that uses [SageMath] for computations 

## ℹ️ Overview

Please refer to [Finding Roots] for the specific blog and the relevant repo(s). If you don't have time TL;DR:

This was an attempt to learn more about node.js based applications and the use of the Newton-Raphson root finding method was convenient. The UI collects the user's instructions (a function, an inital guess for a root, minimum/maximum for the variable, limit of iteration steps, and a SageMath command, usually assuming the variable is `x` $\x = var('x') \$. Then a movie for the iteration steps appears when the calculation is completed. 

### ✍️ Authors

All repos shared in [CurioCopia] are shared under Creative Commons license for others to adopt and use it as they wish.

## 🚀 Usage

1. Generate your own Docker image for [redis-reader], place it in your favorite registry.
2. Generate your own Docker image for [nr-ui], place it in your favorite registry.
3. Generate your own Docker image for [nr-backend-service]. Instantiate it in a compute accessible from the Kubernetes cluster where `nr-ui` will be running.
4. Clone this repo, adjust the essential parameters in [demo], run kustomize and generate resource files.
5. Run the resource files.
6. Access the UI via browser and enjoy.

## ⬇️ Installation

Let's follow the typical Kustomize installation process.

Define a place to work:
```bash
TEST_HOME=$(mktemp -d)
```
### Establish the Base

```bash
BASE=$TEST_HOME/base
mkdir -p $BASE

CONTENT="https://raw.githubusercontent.com/Curiocopia/blog-finding-roots/refs/heads/main"

curl -s -o "$BASE/#1" "$CONTENT/base\
/{externalsagemath-endpointslice.yaml,externalsagemath-service.yaml,kustomization.yaml,redis-pod.yaml,redis-service.yaml,redis-nodeport-service.yaml,nr-deployment.yaml,nr-nodeport-service.yaml,finding-roots.env}"
```
Look at the directory:
```bash
tree $TEST_HOME
```
Expect something like:
```bash
/tmp/tmp.OdCWAqtRU4
└── base

```
### The Base Customization

The base directory has a kustomization file:
```bash
more $BASE/kustomization.yaml
```
You can run kustomize on the base to emit customized resources to stdout and inspect:
```bash
kustomize build $BASE
```
More conveniently you can generate the kustomize output and split the resources into their own files and then execute them in the preferred order.
```bash
RESOURCES=$TEST_HOME/resources
mkdir -p $RESOURCES

kustomize build $BASE -o $RESOURCES

tree $TEST_HOME
/tmp/tmp.OdCWAqtRU4
├── base
│  
└── resources
    ├── 
```
Follow the recipe below (adjust for your own exact filenames) for ConfigMap, Redis Pod and Service and headless SageMath Service and EndPointSlice:
```bash
cd $RESOURCES

```
Once the resources are running, follow the recipe below for filling the Redis queue and processing it.
```bash
kubectl apply -f
```
## Create Overlay

Create a `demo` overlay.
```bash
OVERLAYS=$TEST_HOME/overlays
mkdir -p $OVERLAYS/demo
```
## Demo Customization

```bash
curl -s -o "$OVERLAYS/demo/#1" "$CONTENT/overlays/demo\
/{kustomization.yaml,sagemath-endpointslice-patch.yaml,sagemath-service-patch.yaml,redis-nodeport-service-patch.yaml,nr-nodeport-service-patch.yaml,finding-roots-demo.env}"
```
Adjust the parameters as you need. Set `namespace` for all resources and `nr-ui` and `redis-reader` `image` in `kustomization.yaml`:
```yaml
namespace: demo

images:
- name: nr-ui
  newName: my-registry/nr-ui
  newTag: v1
- name: redis-reader
  newName: my-registry/redis-reader
  newTag: v1
```
Adjust `finding-roots-demo.env` values for ConfigMap creation to use in various reources.

Change the `spec.externalIPs` in the `sagemath-service-patch.yaml` based on the IP address for the `sagemath-backend-service`: 
```yaml
  externalIPs:
  - 192.168.1.200
```
Use the same value for the `endpoints.addresses` in the `sagemath-endpointslice-patch.yaml`: 
```yaml
endpoints:
- addresses:
  - "192.168.1.200"
```
Add the value for the `spec.ports[0].nodePort` in the `redis-nodeport-service-patch.yaml`:
```yaml
spec:
  ports:
  - name: 6379-6379
    port: 6379
    protocol: TCP
    nodePort: 30007
```

Do the same for the `spec.ports[0].nodePort` in the `nr-nodeport-service-patch.yaml`:
```yaml
spec:
  ports:
  - name: 5173-80
    port: 5173
    protocol: TCP
    nodePort: 30100
```
Delete and previous values in the resources and create the new resource files.
```bash
rm $RESOURCES/*
kustomize build $OVERLAYS/demo -o $RESOURCES
```
Inspect the values. If you are satisfied, follow the same base recipe for deployment and execution after you create the `demo` namespace.
## 💭 Feedback and Contributing

If you have any other suggestions for improvements or corrections, please drop a note in Discussions.

[Finding Roots]: https://curiocopia.com/blog/finding-roots
[SageMath]: https://sagemath.org
[Curiocopia]: https://curiocopia.com
[redis-reader]: https://github.com/Curiocopia/oppermann-worker
[nr-ui]: https://github.com/Curiocopia/SageMath-backend-service
[nr-backend-service]: https://github.com/Curiocopia/nr-backend-service
[demo]: overlays/demo/