## Running Locust in the Cluster as a DaemonSet/Deployment

### Node labels and nodeSelector

You will need to label your nodes accordingly. There should be at least 1x Locust coordinating node and 1x Locust worker node.

```bash
kubectl label nodes <YOUR_COORDINATING_NODE> locustRole=coordinator
```

This is specified in the `coordinator-deployment.yaml` template:

```yaml
...
nodeSelector:
  locustRole: coordinator
...
```

Likewise the remaining nodes are labelled as worker nodes:

```bash
kubectl label nodes <WORKER_NODE_1> locustRole=worker
kubectl label nodes <WORKER_NODE_2> locustRole=worker
...
```

And the `worker.yaml` file specifies the following nodeSelector:

```yaml
...
nodeSelector:
  locustRole: worker
...
```

### Helm Chart Install

*Note: I am running my applications in Kubernetes with a LoadBalancer available - you may need to make changes to the chart if you wish to use a different Service type.*

To run Locust in with the DaemonSet configuration the `workers.kind` property in the chart `values.yaml` should be set to `DaemonSet`:

```yaml
...
workers:
  kind: Daemonset
...
```

The chart can then be installed with the following command:

```bash
$ helm install locust-test ./locustchart
$ kubectl port-forward svc/locust-test-coordinator-service-web 8089:80
```

## Running Locust with worker replicas

Alternatively you may run the Locust workers in a Deployment with a number of replicas:

```bash
$ helm install --set workers.kind=Deployment --set workers.replicas=6 locust-test ./locustchart
$ kubectl port-forward svc/locust-test-coordinator-service-web 8089:80
```

## Access web interface

Open http://127.0.0.1:8089/ in a browser and configure a run
