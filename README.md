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

Introduce an insecure web application called ```storefront``` into the cluster:
```
kubectl apply -f https://installer.calicocloud.io/storefront-demo.yaml
```

<img width="947" alt="Screenshot 2024-01-17 at 16 04 49" src="https://github.com/nigel-falco/zapproxy-testing/assets/152274017/48506aaf-66a4-436e-8969-23fdc852990e">

## Zap API Scan

Trying to figure out what the ```zap-api-scan.py``` script does:
```
kubectl exec -it $(kubectl get pod -l app.kubernetes.io/name=owasp-zap -n zap -o jsonpath="{.items[0].metadata.name}") -n zap -- zap-api-scan.py -t http://10.100.244.74:80
```

The synax was modified from a similar example that used Docker for performing tests: <br/>
https://www.fullsecurityengineer.com/headless-web-application-scanning-with-owasp-zap/

Still working on configuring the correct syntax for the PenTest operations

<img width="1429" alt="Screenshot 2024-01-17 at 18 30 30" src="https://github.com/nigel-falco/zapproxy-testing/assets/152274017/339288fe-2b3c-43cb-b124-fc463bf38cce">

## Zap Full Scan

Initially tested the ```-h``` feature flag for the ```zap-full-scan.py``` to ensure I have it correctly configured:
```
kubectl exec -it $(kubectl get pod -l app.kubernetes.io/name=owasp-zap -n zap -o jsonpath="{.items[0].metadata.name}") -n zap -- zap-full-scan.py -h
```

<img width="1268" alt="Screenshot 2024-01-17 at 18 57 06" src="https://github.com/nigel-falco/zapproxy-testing/assets/152274017/a1e9bf9f-5038-49e0-80e5-5187e4018b55">

## Exporting a report

The output of the full scan only prints the status of the checks, but does not tell any findings that the OWASP ZAP found during the assessment.
In order to obtain a report from OWASP ZAP, you may run the following command to write in the current path, a report called ```test_report.html```.

```
kubectl exec -it $(kubectl get pod -l app.kubernetes.io/name=owasp-zap -n zap -o jsonpath="{.items[0].metadata.name}") -n zap -- zap-full-scan.py -g gen-conf -r test_report.html -t http://10.100.244.74:80
```

## Zap.sh

Have not idea what this does, but after describing the pod, I can see that the formatting I was previously using was invalid. I believe I have made some decent progress here.

```
kubectl exec -it $(kubectl get pod -l app.kubernetes.io/name=owasp-zap -n zap -o jsonpath="{.items[0].metadata.name}") -n zap -- zap.sh -daemon -host 0.0.0.0 -port 8081
```

<img width="1268" alt="Screenshot 2024-01-17 at 19 24 02" src="https://github.com/nigel-falco/zapproxy-testing/assets/152274017/8ec993a8-b1c6-48cf-adb0-d512ac7c72bd">

## 2 Concurrent Zap Instances Running
```
kubectl exec -it $(kubectl get pod -l app.kubernetes.io/name=owasp-zap -n zap -o jsonpath="{.items[0].metadata.name}") -n zap -- zap.sh -daemon -port 8090 -host 0.0.0.0
```
```
Found Java version 11.0.21
Available memory: 15802 MB
Using JVM args: -Xmx3950m
The home directory is already in use. Ensure no other ZAP instances are running with the same home directory: /home/zap/.ZAP/
command terminated with exit code 1
```
There is a thread on this issue:
https://groups.google.com/g/zaproxy-users/c/rycqNK0SFRE

I fixed the command to point to a new ```$HOME``` directory:
```
kubectl exec -it $(kubectl get pod -l app.kubernetes.io/name=owasp-zap -n zap -o jsonpath="{.items[0].metadata.name}") -n zap -- zap.sh -daemon -port 8090 -host 0.0.0.0 -dir /home/zap/.ZAP-instance1/
```
Use the ```-dir``` argument to specify a different home. <br/>
https://www.zaproxy.org/docs/desktop/cmdline/#options
<br/><br/>
The homes can't be shared by two ZAP instances running concurrently. <br/>
The ```-dir``` flag uses the specified directory as home directory, instead of the default one.
<br/><br/>
To prevent add-ons (inadvertently) use/override core files ZAP will not start (and show an error) if the home and the installation directories are the same.

<img width="1400" alt="Screenshot 2024-01-19 at 11 46 06" src="https://github.com/nigel-falco/zapproxy-testing/assets/152274017/62e50580-3ec4-49c9-a190-7bdfe0d4379d">

The previous script only initiatates Zap Proxy, but doesn't perform the scan. <br/>
We need to modify this scan command to get it working:
```
kubectl exec -it $(kubectl get pod -l app.kubernetes.io/name=owasp-zap -n zap -o jsonpath="{.items[0].metadata.name}") -n zap -- zap-baseline.py -t http://falco-falcosidekick-ui.falco.svc.cluster.local:2802/ -r /zap/wrk/report.html -d /zap/wrk/
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
```
kubectl port-forward svc/falco-falcosidekick-ui -n falco 2802 --insecure-skip-tls-verify
```
## Scale down the cluster
```
eksctl scale nodegroup --cluster falco-cluster --name ng-201ab6f7 --nodes 0
```
