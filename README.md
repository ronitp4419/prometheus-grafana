# prometheus-grafana
##### Observability with Prometheus and Grafana in K8S

## Kubernetes 1.23 Monitoring Guide

##### Install K8S cluster on local system with Kind
Create a cluster with [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

```
kind create cluster --name monitoring --image kindest/node:v1.23.6 --config kind.yaml
```

Test our cluster to see all nodes are healthy and ready:

```
kubectl get nodes
```

## Kube Prometheus

The best method for monitoring, is to use the community manifests on the `kube-prometheus`
repository [here](https://github.com/prometheus-operator/kube-prometheus)

Now according to the compatibility matrix, we will need `release-0.10` to be compatible with
Kubernetes 1.23. </br>

Let's use docker to grab it!

```
docker run -it -v ${PWD}:/work -w /work alpine sh
apk add git
```

Shallow clone the release branch into a temporary folder:

```
# clone
git clone --depth 1 https://github.com/prometheus-operator/kube-prometheus.git -b release-0.10 /tmp/

# view the files
ls /tmp/ -l

# we are interested in the "manifests" folder
ls /tmp/manifests -l

# let's grab it by coping it out the container
cp -R /tmp/manifests .
```

## Prometheus Operator

To deploy all these manifests, we will need to setup the prometheus operator and custom resource definitions required.

This is all in the `setup` directory:

```
ls /tmp/manifests/setup -l
```

Now that we have the source code manifests, we can exit our temporary container

```
exit
```

## Setup CRDs

Let's create the CRD's and prometheus operator

```
kubectl create -f ./manifests/setup/
```

## Setup Manifests

Apply rest of manifests

```
kubectl create -f ./manifests/
```

## Check Monitoring

```
kubectl -n monitoring get pods
```

## View Dashboards

You can access the dashboards by using `port-forward` to access Grafana.
It does not have a public endpoint for security reasons

```
kubectl -n monitoring port-forward svc/grafana 3000
```

Then access Grafana on [localhost:3000](http://localhost:3000/)

## Fix Grafana Datasource

Now for some reason, the Prometheus data source in Grafana does not work out the box.
To fix it, we need to change the service endpoint of the data source. </br>

To do this, edit `manifests/grafana-dashboardDatasources.yaml` and replace the datasource url endpoint with `http://prometheus-operated.monitoring.svc:9090` </br>

We'll need to patch that and restart Grafana

```
kubectl apply -f ./manifests/grafana-dashboardDatasources.yaml
kubectl -n monitoring delete po <grafana-pod>
kubectl -n monitoring port-forward svc/grafana 3000
```

Now our datasource should be healthy.

## Check Prometheus

Similar to checking Grafana, we can also check Prometheus:

```
kubectl -n monitoring port-forward svc/prometheus-operated 9090
```

## Check Service Monitors

To see how Prometheus is configured on what to scrape , we list service monitors

```
kubectl -n monitoring get servicemonitors

kubectl -n monitoring describe servicemonitor node-exporter
```

Label selectors are used to map service monitor to kubernetes services. </br>

That is how Prometheus is configured on what to scrape.
