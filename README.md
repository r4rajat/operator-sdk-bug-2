# operator-sdk bug

Sample app to demo a Red Hat `operator-sdk` bug on OpenShift.

Bug Report on Upstream: https://github.com/operator-framework/operator-sdk/issues/

## Environment

```
Openshift Version: OpenShift v4.12.25
```
```
Kubectl Version: Client Version: v1.28.0
                 Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
                 Server Version: v1.25.11+1485cc9
                 WARNING: version difference between client (1.28) and server (1.25) exceeds the supported minor version skew of +/-1
```
```
Operator SDK Version: operator-sdk version: "v1.31.0", commit: "e67da35ef4fff3e471a208904b2a142b27ae32b1", kubernetes version: "1.26.0", go version: "go1.19.11", GOOS: "linux", GOARCH: "amd64"
```

## Summary

This is a Sample `hello-world` helm chart with a deployment. It also has a child helm chart `prometheus`, which again contains 
4 sub-charts i.e `alertmanager`, `kube-state-metrics`, `prometheus-node-exporter` and `prometheus-pushgateway`. which are `disabled` from `prometheus`'s [values.yaml](https://github.com/r4rajat/operator-sdk-bug-2/blob/main/helm/hello-world/charts/prometheus/values.yaml#L1158-L1204).

After installing operator and operand on the cluster, we could see the sub-charts `alertmanager`, `kube-state-metrics`, 
`prometheus-node-exporter` and `prometheus-pushgateway` have also been installed despite being disabled from `prometheus`'s [values.yaml](https://github.com/r4rajat/operator-sdk-bug-2/blob/main/helm/hello-world/charts/prometheus/values.yaml#L1158-L1204).

## Steps Followed

```bash
$ operator-sdk init \
--project-name=hello-world-operator --plugins=helm --domain=dev \
--helm-chart=./helm/hello-world --verbose
```

```bash
$ make docker-build docker-push
```

```bash
$ operator-sdk olm install
```

```bash
$ make bundle bundle-build bundle-push
```

```bash
$ operator-sdk run bundle docker.io/r4rajat/hello-world-operator-bundle:v1.0.1 --timeout 15m
```

```bash
$ kubectl apply -f ./config/samples/charts_v1alpha1_helloworld.yaml
```