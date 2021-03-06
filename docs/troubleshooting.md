# Troubleshooting

This document contains suggestions for debugging issues with your Contour installation.

## Envoy container not listening on port 8080 or 8443

Contour does not configure Envoy to listen on a port unless there is traffic to be served.
For example, if you have not configured any TLS ingress objects then Contour does not command Envoy to open port 8443 (443 in the service object).
Because the HTTP and HTTPS listeners both use the same code, if you have no ingress objects deployed in your cluster, or if no ingress objects are permitted to talk on HTTP, then Envoy does not listen on port 8080 (80 in the service object).

To test whether Contour is correctly deployed you can deploy the kuard example service:
```
$ kubectl apply -f https://j.hept.io/contour-kuard-example
```

## Access the Envoy admin interface remotely

Getting access to the Envoy admin interface can be useful for diagnosing issues with routing or cluster health.

The Envoy admin interface is bound by default to `http://127.0.0.1:9001`. 
To access it from your workstation use `kubectl port-forward` like so,
```
# Get one of the pods that matches the deployment/daemonset
CONTOUR_POD=$(kubectl -n heptio-contour get pod -l app=contour -o jsonpath='{.items[0].metadata.name}')
# Do the port forward to that pod
kubectl -n heptio-contour port-forward $CONTOUR_POD 9001
```
Then navigate to [http://127.0.0.1:9001/](http://127.0.0.1:9001/) to access the admin interface for the Envoy container running on that pod.

## Can't make kube-lego work with Contour

If you use [kube-lego][0] for Let's Encrypt SSL certificates, kube-lego appears to set the ingress class on the ingress record it uses for the acme-01 challenge to `nginx`.
This setting causes Contour to ignore the record, and the challenge fails.

The current workaround is to manually edit the ingress object created by kube-lego to remove the `kubernetes.io/ingress.class` annotation. Or you can set the value of the annotation to `contour`.
If your topology allow it, you may configure Contour to catch `nginx` ingress class by overriding the default value with the `--ingress-class-name=nginx` flag in your Contour deployment.

[0]: https://github.com/jetstack/kube-lego
