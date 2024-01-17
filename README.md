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

<img width="1036" alt="Screenshot 2024-01-17 at 10 52 45" src="https://github.com/nigel-falco/zapproxy-testing/assets/152274017/6e5b5af9-c154-4678-9356-bee396e299c4">


You can then view the exposed webpage at http://localhost:8081

<img width="1204" alt="Screenshot 2024-01-17 at 10 53 31" src="https://github.com/nigel-falco/zapproxy-testing/assets/152274017/f0a12f3d-7bfc-47d7-870f-7b70198f8625">

Terminal shell into the Zap Proxy pod:
```
kubectl exec -it -n zap deploy/zap-owasp-zap -- bash
```

<img width="1056" alt="Screenshot 2024-01-17 at 12 16 17" src="https://github.com/nigel-falco/zapproxy-testing/assets/152274017/8a8a3550-7d79-4e0c-be7c-175d6c0e08b6">



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
