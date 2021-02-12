# kube-signal-controller

Annotate K8s pods to have signals (SIGHUP, etc) sent on a schedule

## Example

In this example showcasing defaults, the `SIGHUP` signal is sent to PID 1 of all containers in the pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  annotations:
    pbar.me/signal-schedule: 0 18 * * *
spec:
  containers:
  - name: container1
    image: nginx
  - name: container2
    image: nginx
```

Schedule and target configuration may also be specified in a more granular way, as seen here:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
  annotations:
    pbar.me/signal-config: |
      - container: container1
        process: nginx
        signal: SIGHUP
        schedule: 0 18 * * *
      - container: container2
        process: 1
        signal: SIGTERM
        schedule: @weekly
spec:
  containers:
  - name: container1
    image: nginx
  - name: container2
    image: nginx
```

## Methods of sending the signal

### `kubectl exec`

Pros:
- Works on all clusters using standard features
- Works regardless of process namespace configuration on the pod

Cons:
- Only works on container images with userspace tooling. Not Distroless, etc

### `kubectl debug`

Pros:
- Works with any container image, even Distroless

Cons:
- Currently in alpha, must be enabled via a Kubernetes feature gate
- Could pollute pod manifest with EphemeralContainers

### Sidecar container

Pros:
- Works on all clusters using standard features
- Works with any container image, even Distroless

Cons:
- Sidecar container must be injected
- Sidecar container must share process namespace with target container
