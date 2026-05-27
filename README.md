# EnderDash deploy

Public deployment manifests and examples for the EnderDash standalone agent.

Use the generated agent key from your EnderDash server setup page wherever the
examples use `<agentKey>`.

## Docker Compose

Download the Compose file, set the agent key, and start the agent:

```bash
curl -fsSLO https://raw.githubusercontent.com/enderdash-com/deploy/main/agent/compose.yaml
export ENDERDASH_AGENT_KEY='<agentKey>'
export DOCKER_SOCKET_GID="$(stat -c '%g' /var/run/docker.sock)"
docker compose up -d
```

## Podman Quadlet

Download the Quadlet files into your user systemd unit directory:

```bash
mkdir -p ~/.config/containers/systemd
curl -fsSL https://raw.githubusercontent.com/enderdash-com/deploy/main/agent/quadlet/enderdash-agent.container \
  -o ~/.config/containers/systemd/enderdash-agent.container
curl -fsSL https://raw.githubusercontent.com/enderdash-com/deploy/main/agent/quadlet/enderdash-agent-data.volume \
  -o ~/.config/containers/systemd/enderdash-agent-data.volume
```

Write the persistent environment file, then reload and start the units:

```bash
cat > ~/.config/containers/systemd/enderdash-agent.env <<'EOF'
ENDERDASH_AGENT_KEY=<agentKey>
EOF
chmod 600 ~/.config/containers/systemd/enderdash-agent.env

systemctl --user enable --now podman.socket
systemctl --user daemon-reload
systemctl --user enable --now enderdash-agent.service
```

## Kubernetes Kustomize

Create the namespace and agent key Secret:

```bash
kubectl create namespace enderdash --dry-run=client -o yaml | kubectl apply -f -
kubectl -n enderdash create secret generic enderdash-agent \
  --from-literal=agentKey='<agentKey>' \
  --dry-run=client -o yaml | kubectl apply -f -
```

Install the read-only manifests:

```bash
kubectl apply -k 'https://github.com/enderdash-com/deploy//agent/kustomize/base?ref=main'
```

Use operator mode only for clusters where EnderDash should run Kubernetes
mutations such as restarts, scaling, exec, port-forward, debug containers, and
YAML apply or delete actions:

```bash
kubectl apply -k 'https://github.com/enderdash-com/deploy//agent/kustomize/operator?ref=main'
```

## Helm

Helm charts are published separately:

```bash
helm repo add enderdash https://charts.enderdash.com
helm repo update
helm upgrade --install enderdash-agent enderdash/enderdash-agent \
  --namespace enderdash \
  --create-namespace \
  --set agentKeySecret.name=enderdash-agent \
  --set agentKeySecret.key=agentKey \
  --set rbac.mode=readonly
```
