# Istio_ingressGateway

# 1. Make Istio Ingress gateway 
```
git clone https://github.com/developer-onizuka/Istio_ingressGateway
cd Istio_ingressGateway
kubectl apply -f ingress-gateway.yaml
```

# 2. Service of Cat
```
cd cat
kubectl create configmap nginx-cat-html --from-file=index.html
kubectl apply -f nginx_cat.yaml
cd -
```


# 3. Service of Dog
```
cd dog
kubectl create configmap nginx-dog-html --from-file=index.html
kubectl apply -f nginx_dog.yaml
cd -
```
