# p46-cronjobs

A Helm chart that deploys a Kubernetes CronJob to track when PersistentVolumeClaims (PVCs) in the `p46-beamline` namespace were last used. This supports automated cleanup of old, unused PVCs.

## What it does

Every 5 minutes the `last-used-stamper` job:

1. Lists all PVCs in `p46-beamline`
2. Lists all PVCs currently mounted by a running pod
3. For each PVC, annotates it with the current unix timestamp (`last-used`) if:
   - It has never been annotated before, or
   - It is currently mounted (i.e. actively in use)

PVCs that are unmounted and already have an annotation are left untouched, preserving the timestamp of when they were last used.

## Annotation

| Key | Format | Example |
|-----|--------|---------|
| `last-used` | Unix timestamp (seconds) | `1742900000` |

Any Kubernetes Job in the cluster can read this annotation with:

```bash
kubectl get pvc <name> -n p46-beamline -o jsonpath='{.metadata.annotations.last-used}'
```

## RBAC

The chart creates a `ClusterRole` and `ClusterRoleBinding` granting the CronJob's ServiceAccount the following permissions across all namespaces:

| Resource | Verbs |
|----------|-------|
| `pods` | `get`, `list` |
| `persistentvolumeclaims` | `get`, `list`, `patch` |

A `ClusterRole` is used (rather than a namespaced `Role`) so that the permissions cover the `p46-beamline` namespace regardless of which namespace this chart is deployed into.
