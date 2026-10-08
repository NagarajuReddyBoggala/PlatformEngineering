# Platform Engineering — Mistake & Weak-Area Log

## Docker

- `EXPOSE` does **not** publish a port. `-p` publishes the port.
- `docker rm` removes a container; `docker rmi` removes an image.
- `127.0.0.1` inside a container refers to the container itself.
- Backend containers should commonly listen on `0.0.0.0`, not only `127.0.0.1`.
- `RUN` happens during image build; `CMD` runs as the default container command.
- `ENTRYPOINT` defines the main executable.
- Better cache usage requires copying dependency files before application source.
- `.dockerignore` prevents unnecessary files from entering the build context.
- Multi-stage builds are useful when build dependencies are not required at runtime.
- `docker compose down` does not normally remove named volumes unless `--volumes` is used.

## Kubernetes

- Deployment does not directly maintain individual Pods; ReplicaSet does.
- `kube-controller-manager` contains multiple controllers, including Deployment and ReplicaSet controllers.
- Scheduler decides **where** a Pod runs; kubelet makes the assigned Pod run.
- Pod resource **requests** matter for scheduling; current usage from `k top` is different.
- A toleration allows a Pod to tolerate a taint; it does not force scheduling onto that node.
- A Service needs matching Pod labels through its selector to create endpoints.
- `targetPort` must match the port where the application is actually listening.
- Service routing path is different from application path; Ingress rewrite may be required.
- `k get endpoints` works for inspection, but EndpointSlice is the modern API.
- `CrashLoopBackOff` means the container repeatedly starts and crashes; the Pod may remain the same.
- `ImagePullBackOff` means Kubernetes is backing off repeated failed image pulls.
- `Pending` means the Pod has not successfully reached the running stage; inspect scheduling/events.
- Directly-created Pods are not managed like Deployment Pod templates.
- `envFrom` can consume ConfigMaps or Secrets, not only ConfigMaps.
- `env` can reference specific ConfigMap/Secret keys.
- PVC requests storage; StorageClass enables dynamic provisioning; PV represents the provisioned storage.
- Role is namespace-scoped; ClusterRole can be cluster-wide/reusable.
- NetworkPolicy is enforced by the CNI/network implementation, not directly by the API server.
