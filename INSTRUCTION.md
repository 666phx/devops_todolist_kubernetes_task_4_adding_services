# Instruction

## 0. Deploy manifests

```
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/todoapp-pod.yml
kubectl apply -f .infrastructure/cluster-ip.yml
kubectl apply -f .infrastructure/nodeport.yml
kubectl apply -f .infrastructure/busybox.yml
```

Check that both `todoapp` pods are `Running` and `Ready`:

```
kubectl get pods -n todoapp
```

## 1. Test the app by calling the ClusterIP service DNS from a busybox container

Exec into the `busybox` pod (already deployed in the `todoapp` namespace and includes `curl`):

```
kubectl exec -it busybox -n todoapp -- sh
```

Inside the container, call the ClusterIP service by its DNS name (short name works because busybox is in the same namespace; the full form is `<service>.<namespace>.svc.cluster.local`):

```
curl todo-clusterip-service
# or
curl todo-clusterip-service.todoapp.svc.cluster.local
```

A successful response (HTML of the landing page or a JSON response from `/api/`) confirms the ClusterIP service is balancing traffic between the two `todoapp` pods. Run the `curl` command a few times in a row — requests should be served by different pods.

## 2. Test the ToDo application using the `port-forward` command

Forward a local port to the ClusterIP service:

```
kubectl port-forward svc/todo-clusterip-service 8080:80 -n todoapp
```

Open the app in a browser (or with `curl`) on the machine running `kubectl`:

```
http://localhost:8080/
http://localhost:8080/api/
```

Stop forwarding with `Ctrl+C` when done.

## 3. Access the app using the NodePort Service

Get the IP address of any cluster node:

```
kubectl get nodes -o wide
```

(If you are using Minikube, get the IP with `minikube ip`.)

The NodePort service exposes the app on port `30007` on every node. Open in a browser or with `curl`:

```
http://<NODE_IP>:30007/
http://<NODE_IP>:30007/api/
```
