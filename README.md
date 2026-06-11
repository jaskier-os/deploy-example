# deploy-example

> **Docs & wiki:** [github.com/jaskier-os/docs/wiki](https://github.com/jaskier-os/docs/wiki)

A static example of the Kubernetes deployment catalog for the jaskier-os stack. Use it as a
reference for what manifests the stack needs and how they fit together: Secrets are
`*.yaml.example` files you copy and fill in, hosts are tokenized, and image tags are `:latest`.

This is a reference, not a live GitOps source. Apply it with plain `kubectl`/`kustomize`.

## Layout

```
apps/
  ai/        backend services + agents (namespace: ai)
  recon/     ReID pipeline + mongodb + flaresolverr (namespace: recon)
infrastructure/
  namespaces.yaml        ai + recon namespaces
  secrets/*.yaml.example placeholder Secrets (copy + fill, or create out-of-band)
  tls/                   cert-manager ClusterIssuer + server Certificate
  backup/                mongodb backup CronJob
```

## What is included

- ai: orchestrator, communicator, anthropic-stt, kokoro-tts, piper-tts, teratts-tts,
  translator, ocr, and the agents (vision, web-search + mcp-web-search, reid, clickup,
  security, chat-history).
- recon: reid-worker, reid-db-handler, reid-analytics-backend, mongodb, flaresolverr.

## Placeholders to substitute

- Image: `localhost:5000/<svc>:latest` -> point at your registry/tag.
- Hosts: `EXTERNAL_HOST`, `RECON_HOST`, `GITLAB_HOST`, `CLOUD_HOST` -> your addresses.
- Secrets: `CHANGE_ME` / `REPLACE_AT_DEPLOY` -> real values (never commit them).
- TLS: the server Certificate's SANs -> your hostnames/IPs.

## Deploy

```bash
# 1. namespaces + TLS + backup
kubectl apply -k infrastructure/

# 2. secrets (NOT applied by kustomize): copy each example, fill in, and apply --
#    or create them directly, e.g.
cp infrastructure/secrets/ai-secrets.yaml.example /tmp/ai-secrets.yaml   # edit values
kubectl apply -f /tmp/ai-secrets.yaml
#    repeat for recon-secrets and (optionally) communicator-refresh-ssh.

# 3. workloads
kubectl apply -k apps/ai/
kubectl apply -k apps/recon/

# 4. verify
kubectl get pods -n ai
kubectl get pods -n recon
```

Per-service detail (build, env vars, ports) lives in each service's own repo; the system
overview lives in jaskier-os/docs.
