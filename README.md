# Configure-a-external-loadbalancer-to-cluster-
## Create Deployment
```bash
vi app-deployment.yml
```
```bash
# app-deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
```bash
kubectl apply -f app-deployment.yml
```
## Create NodePort Service
```bash
vi app-service.yml
```
```bash
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```
```bash
kubectl apply -f app-service.yml
```
# STEP 2 — Install HAProxy on (RHEL 9 or any linux os)

```bash
 dnf install haproxy -y
```
```bash
 systemctl enable haproxy
```
```bash
 vi /etc/haproxy/haproxy.cfg
```
## Replace config with this clean working config:
```
global
    log /dev/log local0
    daemon

defaults
    mode http
    timeout connect 5s
    timeout client  50s
    timeout server  50s

frontend kubernetes-frontend
    bind *:80
    default_backend kubernetes-backend

backend kubernetes-backend
    balance roundrobin
    server worker1 192.168.56.11:30007 check ## replacae with cluster worker-node ip
    server worker2 192.168.56.12:30007 check ## replacae with cluster worker-node ip
```
## Stop firewall service if you do not stop firewall servcie then add servcie inside firewall ip tables

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```
## putt selinux in permissive mode

```bash
setenforce 0
```
## Start HAProxy

```bash
sudo systemctl start haproxy
sudo systemctl status haproxy
```
