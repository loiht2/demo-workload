# demo-workload

A minimal Kubernetes workload used to exercise [opsgentic](https://github.com/lehuannhatrang/opsgentic) end to end: trigger an alert, run RCA + validation, and receive a remediation pull request against this repo.

## What's here

```
apps/payments-api/      # the demo app (HighPodMemory remediation target)
  namespace.yaml        # namespace: payments
  deployment.yaml       # tight memory limit (64Mi) -> fires HighPodMemory
  service.yaml          # placeholder Service
  kustomization.yaml
monitoring/
  prometheusrule.yaml   # optional HighPodMemory alert (Prometheus Operator)
```

The `payments-api` pod allocates ~50Mi against a 64Mi limit (~80%), so it sits above the alert threshold without being OOM-killed. The intended remediation is to raise `resources.limits.memory`.

## Deploy

```bash
kubectl apply -k apps/payments-api/
# optional, requires kube-prometheus-stack:
kubectl apply -f monitoring/prometheusrule.yaml
```

Check it is running near its limit:

```bash
kubectl -n payments get pods
kubectl -n payments top pod      # requires metrics-server
```

## Trigger opsgentic

- **Real alert path:** point Alertmanager/Grafana at opsgentic's `POST /webhook/grafana`. The `HighPodMemory` alert carries `gitops_repo` and `gitops_path` labels so the remediation PR targets `apps/payments-api/deployment.yaml`.
- **Manual test:** post a synthetic alert or chat message to opsgentic (see opsgentic's `examples/grafana_alert.json` and `examples/chat_input.json`, already pointed at this repo).

opsgentic runs RCA → validation → pauses for human approval → opens a remediation-proposal PR here. Merge it (and let ArgoCD/Flux sync, or apply manually) to clear the alert.
