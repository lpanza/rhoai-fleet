# fake-gpu (lab only)

Simulated NVIDIA GPUs with [run-ai/fake-gpu-operator](https://github.com/run-ai/fake-gpu-operator)
0.2.1: one node advertises `nvidia.com/gpu: 8` as `NVIDIA-H100-80GB-HBM3`. Pods
can request GPUs and get a simulated `nvidia-smi`, but there is no real compute.

Mutually exclusive with the real NVIDIA GPU Operator: never set both on a cluster.

## Enable on a cluster (hub)

```bash
oc label managedcluster cluster-a \
  ai.fleet/gpu=fake \
  ai.fleet/fake-gpu-node=ip-10-0-25-39.us-east-2.compute.internal
```

Requires `ai.fleet/rhoai=true`. Removing `ai.fleet/gpu` deletes everything the
policy created (`pruneObjectBehavior: DeleteIfCreated`); the node label
`run.ai/simulated-gpu-node-pool` stays on the node and must be removed by hand.

## Change the simulation

Edit `chart/values.yaml` (GPU model, count, memory), then:

```bash
./render.sh        # needs helm, yq; rewrites manifests/fake-gpu-operator.yaml
git diff manifests/
```

`render.sh` drops chart objects that don't belong on OpenShift or that would
collide with the real GPU operator: a fake operator CSV in
`openshift-operator-lifecycle-manager`, the `clusterlogging-collector-metrics`
RBAC, the `nvidia` RuntimeClass, the NVIDIA CRDs and the KWOK components.
