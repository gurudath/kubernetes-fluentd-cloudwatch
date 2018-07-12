# Kubernetes + Fluentd + Cloudwatch

This is a modified version of [fluentd](https://www.fluentd.org) to easily integrate with EKS and send logs to AWS Cloudwatch.

### Requirements

- aws-cli
- kubectl
- helm
- minikube (optional)

## Setup

1. First create a log group, you can do this via the cli or on in Cloudwatch under AWS services.

> `aws logs create-log-group --log-group-name logs`

2. Make sure you are connected to kubernetes and that you have helm tiller installed. You'll run this command or you can edit the `values.yaml` file with your AWS secrets and the log group name.

> `helm install fluentd-cloudwatch/ --set aws.access_key="id" --set aws.secret_key="secret" --set aws.loggroup="logs" --name fluentd


3. You can verify by checking the log group on the website.

## Updating Chart

1. Update the `fluentd-cloudwatch/Chart.yaml` file to the desired version then run following commands. Remove old packages.

```bash
helm package fluentd-cloudwatch/
helm repo index .
```

## Troubleshoot

#### Known error in fluentd `Aws::CloudWatchLogs::Errors::InvalidSignatureException"` it happens mostly with minikube when the clock is out of sync. **Fix**: 

- `minikube ssh -- docker run -i --rm --privileged --pid=host debian nsenter -t 1 -m -u -n -i date -u $(date -u +%m%d%H%M%Y)`

#### If logs aren't showing up on cloudwatch, you can view the fluentd logs via kubectl.

- `kubectl --namespace kube-system logs pod-name`

#### You can exec into fluentd to make changes and restart without having to manually edit everything and rebuild the container.

-  `kubectl --namespace kube-system exec -it fluentd-cloudwatch-287nl -- /bin/sh
exec fluentd -c /fluentd/etc/$FLUENTD_CONF -p /fluentd/plugins $FLUENTD_OPT`

## Resources

- [Fluentd](https://github.com/fluent/fluentd-kubernetes-daemonset)
- [Fluentd image to send Kubernetes logs to CloudWatch](https://github.com/callstats-io/fluentd-kubernetes-cloudwatch)
- [Fluentd Source](https://github.com/fluent/fluentd-kubernetes-daemonset/tree/master/docker-image/v0.12/alpine-cloudwatch)
