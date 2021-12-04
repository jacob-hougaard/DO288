# Pods

## How to understand pod definitions

```bash
apiVersion: v1 // 
kind: Pod // Type af resource
metadata: //
  name: hello-world-pod // navn på vores resource
  labels:
    app: hello-world-pod
spec:
  containers:
  - env: // enviornment variabler
    - name: MESSAGE
      value: Hi! I'm an environment variable
    image: quay.io/practicalopenshift/hello-world // image vi vil pulle fra quay
    imagePullPolicy: Always // 
    name: hello-world-override
    resources: {}
```



## How to use oc explain on pod yaml fields

