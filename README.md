# EnderDash deploy

Public deployment files for the EnderDash standalone agent.

Before you begin, copy the agent key from your EnderDash server setup page.
Replace `<agentKey>` in the examples with that value.

## Docker Compose

To run the agent with Docker Compose:

```bash
curl -fsSLO https://raw.githubusercontent.com/enderdash-com/deploy/main/agent/compose.yaml
cat > .env <<'EOF'
ENDERDASH_AGENT_KEY=<agentKey>
EOF
docker compose up -d
```

To check the container:

```bash
docker compose ps
```

## Podman Quadlet

To run the agent as a rootless Podman systemd user service:

```bash
mkdir -p ~/.config/containers/systemd
curl -fsSL https://raw.githubusercontent.com/enderdash-com/deploy/main/agent/quadlet/enderdash-agent.container \
  -o ~/.config/containers/systemd/enderdash-agent.container
curl -fsSL https://raw.githubusercontent.com/enderdash-com/deploy/main/agent/quadlet/enderdash-agent-data.volume \
  -o ~/.config/containers/systemd/enderdash-agent-data.volume

cat > ~/.config/containers/systemd/enderdash-agent.env <<'EOF'
ENDERDASH_AGENT_KEY=<agentKey>
EOF
chmod 600 ~/.config/containers/systemd/enderdash-agent.env

systemctl --user enable --now podman.socket
systemctl --user daemon-reload
systemctl --user enable --now enderdash-agent.service
```

To check the service:

```bash
systemctl --user status enderdash-agent.service
```

## Kubernetes Kustomize

Create the namespace and Secret:

```bash
kubectl create namespace enderdash
kubectl -n enderdash create secret generic enderdash-agent \
  --from-literal=agentKey='<agentKey>'
```

Install the manifests:

```bash
kubectl apply -k 'https://github.com/enderdash-com/deploy//agent/kustomize/base?ref=main'
```

The default install grants EnderDash cluster-wide Kubernetes permissions so it
can manage workloads, exec sessions, port-forwards, debug containers, and YAML
apply or delete operations.

Install the read-only overlay instead when you want to restrict EnderDash to
inventory and log access:

```bash
kubectl apply -k 'https://github.com/enderdash-com/deploy//agent/kustomize/readonly?ref=main'
```

To check the deployment:

```bash
kubectl -n enderdash get pods -l app.kubernetes.io/name=enderdash-agent
```

## Helm

The Helm chart is published from the
[`enderdash-com/helm-charts`](https://github.com/enderdash-com/helm-charts)
repository.

```bash
helm repo add enderdash https://charts.enderdash.com
helm repo update
helm install enderdash-agent enderdash/enderdash-agent \
  --namespace enderdash
```
