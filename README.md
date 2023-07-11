# Kubernetes rules
Kubernetes rules for the Please build system

## Basic usage

Check out the [codelab](https://please.build/codelabs/k8s) for a step by step guide. 

To add the k8s rules to your project, run `plz init plugin k8s`. You likely want to add the 
[docker](https://github.com/please-build/docker-rules) rules as well with `plz init plugin docker`. Given the following 
docker image:

Dockerfile:
```dockerfile
FROM ubuntu:22.04

COPY /hello_service /hello_service
ENTRYPOINT [ "/hello_service" ]
```

You can build this with:
```python
docker_image(
    name = "image",
    srcs = ["//hello_service"],
    dockerfile = "Dockerfile",
)
```

You can then reference this in a `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  labels:
    app: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: main
          image: //hello_service/k8s:image # This will be replaced by Please with the correct image and tag
          ports:
            # This must match the port we start the server on in hello-service
            - containerPort: 8000
```

And then these rules will replace the image with the docker image we just built like so:

```python

k8s_config(
    name = "k8s",
    srcs = [
        "deployment.yaml",
        "service.yaml",
    ],
    containers = [":image"], # references the image above
)
```

You can then apply these images to a kubernetes cluster like so:
```
$ plz run //hello_service/k8s:image_load && plz run //hello_service/k8s:k8s_push
```
