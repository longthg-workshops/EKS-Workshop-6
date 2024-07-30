---
title: "Ingress"
date: "`r Sys.Date()`"
weight: 15
chapter: false
pre: "<b> 1.15 </b>"
---

#### Ingress

Trong phần này, chúng ta sẽ xem xét **Ingress**

- Bộ điều khiển Ingress
- Tài nguyên Ingress

#### Bộ điều khiển Ingress

- Triển khai **Bộ điều khiển Ingress**

#### Bản đồ Cấu hình

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
```

#### Triển khai

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```

#### Tài khoản Dịch vụ (ServiceAccount)

- ServiceAccount yêu cầu cho mục đích xác thực cùng với các Role, ClusterRole và RoleBinding chính xác.

- Tạo một tài khoản dịch vụ ingress
```
$ kubectl create -f ingress-sa.yaml
serviceaccount/ingress-serviceaccount created
```

#### Loại Dịch vụ - NodePort

```
# service-Nodeport.yaml

apiVersion: v1
kind: Service
metadata:
  name: ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```

- Tạo một dịch vụ
```
$ kubectl create -f service-Nodeport.yaml
```
- Để lấy dịch vụ

```
$ kubectl get service
```

## Tài nguyên Ingress

```
Ingress-wear.yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
     backend:
        serviceName: wear-service
        servicePort: 80
```

- Để tạo tài nguyên ingress
```
$ kubectl create -f Ingress-wear.yaml
ingress.extensions/ingress-wear created
```

- Để lấy ingress
```
$ kubectl get ingress
NAME           CLASS    HOSTS   ADDRESS   PORTS   AGE
ingress-wear   <none>   *                 80      18s
```

#### Tài nguyên Ingress - Luật

- 1 Luật và 2 Đường dẫn.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 80
      - path: /watch
        backend:
          serviceName: watch-service
          servicePort: 80
```
- Mô tả tài nguyên ingress đã tạo trước đó

```
$ kubectl describe ingress ingress-wear-watch
Name:             ingress-wear-watch
Namespace:        default
Address:
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /wear    wear-service:80 (<none>)
              /watch   watch-service:80 (<none>)
Annotations:  <none>
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  23s   nginx-ingress-controller  Ingress default/ingress-wear-watch

```

- 2 Luật và 1 Đường dẫn mỗi luật.
```
# Ingress-wear-watch.yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: watch-service
          servicePort: 80
```

#### Tài liệu Tham khảo

- https://kubernetes.io/docs/concepts/services-networking/ingress/
- https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/
- https://thenewstack.io/kubernetes-ingress-for-beginners/