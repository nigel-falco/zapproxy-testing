# Zap Proxy Testing

Set up an ```AWS-CLI Profile``` in order to interact with AWS services via my local workstation
```
aws configure --profile nigel-aws-profile
export AWS_PROFILE=nigel-aws-profile                                            
aws sts get-caller-identity --profile nigel-aws-profile
aws eks update-kubeconfig --region eu-west-1 --name falco-cluster
```

## Create Kubernetes cluster on AWS

```
eksctl create cluster --name falco-cluster --node-type t3.xlarge --nodes 1 --nodes-min=0 --nodes-max=3 --max-pods-per-node 58
```

Deploy OWASP's Zed Attack Proxy (ZAP) in Kubernetes:
```
kubectl apply -f https://raw.githubusercontent.com/nigel-falco/zapproxy-testing/main/zap-deploy.yaml
```

```
kubectl port-forward service/zap-owasp-zap 8081:8081 -n zap
```

Check that Zap Proxy is running correctly:
```
kubectl get events -n zap
```

## Install Falco and FalcoSideKick

```
helm install falco falcosecurity/falco --namespace falco \
  --create-namespace \
  --set tty=true \
  --set falcosidekick.enabled=true \
  --set falcosidekick.webui.enabled=true \
  --set falcosidekick.webui.redis.storageEnabled=false \
  --set falcosidekick.config.webhook.address=http://falco-talon:2803 \
  --set "falcoctl.config.artifact.install.refs={falco-rules:2,falco-incubating-rules:2,falco-sandbox-rules:2}" \
  --set "falcoctl.config.artifact.follow.refs={falco-rules:2,falco-incubating-rules:2,falco-sandbox-rules:2}" \
  --set "falco.rules_file={/etc/falco/falco_rules.yaml,/etc/falco/falco-incubating_rules.yaml,/etc/falco/falco-sandbox_rules.yaml,/etc/falco/rules.d}"
```

## Scale down the cluster
```
eksctl scale nodegroup --cluster falco-cluster --name <node-group> --nodes 0
```
