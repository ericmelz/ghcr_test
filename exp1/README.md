# nginx with paths (e.g., emelz.com/svc1)

There are 3 docker images, each a simple nginx image that displays
different messages when hit over http.

These containers should be run in kubernetes.
Traefik ingress is used with middleware to strip off path prefixes.

There is a nginx conf file intended to be used in a docker container
running as a reverse proxy.

Push to this github repo to trigger build & push of all images.

# Start a k3d cluster:

```
k3d cluster create --agents 1 -p "8081:80@loadbalancer"
```

# Deploy middleware and services to k3d:
```
kubectl apply -f k8s
```

# Run dockerized nginx with reverse-proxying conf:
```
docker run -p 80:80 -v $(pwd)/nginx/nginx.conf:/etc/nginx/nginx.conf:ro -d --name nginx nginx
```

# Test
```
curl localhost
curl localhost/svc1
curl localhost/svc2
curl localhost/svc3
```

# Troubleshooting:
Open up separate windows for each service and tail the logs

Terminal 1
```
docker logs -f nginx
```
Terminal 2
```
export KUBECONFIG=/Users/ericmelz/.config/k3d/kubeconfig-k3s-default.yaml
kubectl logs -f $(kubectl get po|grep nginx|cut -d' ' -f1)
```

Terminal 1
```
export KUBECONFIG=/Users/ericmelz/.config/k3d/kubeconfig-k3s-default.yaml
kubectl logs -f $(kubectl get po|grep svc1|cut -d' ' -f1)
```
Terminal 2
```
export KUBECONFIG=/Users/ericmelz/.config/k3d/kubeconfig-k3s-default.yaml
kubectl logs -f $(kubectl get po|grep svc2|cut -d' ' -f1)
```
Terminal3
```
export KUBECONFIG=/Users/ericmelz/.config/k3d/kubeconfig-k3s-default.yaml
kubectl logs -f $(kubectl get po|grep svc2|cut -d' ' -f1)
```

# Cleanup
```
# rm docker nginx
docker stop nginx
docker rm nginx

# rm k8s objects
kubectl delete svc --all
kubectl delete deploy --all
kubectl delete ingress --all
kubectl delete middleware --all

# destroy cluster
k3d cluster delete
```