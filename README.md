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
```
# kubectl exec -it dnsutils -- nslookup nginx-cat-svc
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	nginx-cat-svc.default.svc.cluster.local
Address: 10.101.48.11
```

# 3. Service of Dog
```
cd dog
kubectl create configmap nginx-dog-html --from-file=index.html
kubectl apply -f nginx_dog.yaml
cd -
```
```
# kubectl exec -it dnsutils -- nslookup nginx-dog-svc
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	nginx-dog-svc.default.svc.cluster.local
Address: 10.97.229.181
```

# 4. Check Istio gateway created for each service
```
# kubectl get virtualservices
NAME             GATEWAYS              HOSTS   AGE
nginx-cat-vsvc   ["animals-gateway"]   ["*"]   27d
nginx-dog-vsvc   ["animals-gateway"]   ["*"]   27d

# kubectl get services -n istio-system
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                                      AGE
grafana                ClusterIP      10.104.160.57    <none>            3000/TCP                                     4d
istio-ingressgateway   LoadBalancer   10.109.196.134   192.168.121.220   15021:32009/TCP,80:31573/TCP,443:30508/TCP   4d
istiod                 ClusterIP      10.106.237.193   <none>            15010/TCP,15012/TCP,443/TCP,15014/TCP        4d
jaeger-collector       ClusterIP      10.107.120.18    <none>            14268/TCP,14250/TCP,9411/TCP                 4d
kiali                  LoadBalancer   10.104.102.182   192.168.121.222   20001:31219/TCP,9090:31276/TCP               4d
prometheus             ClusterIP      10.100.172.120   <none>            9090/TCP                                     4d
tracing                ClusterIP      10.108.229.244   <none>            80/TCP,16685/TCP                             4d
zipkin                 ClusterIP      10.97.40.27      <none>            9411/TCP                                     4d

# kubectl get pod -n istio-system -o wide
NAME                                    READY   STATUS    RESTARTS       AGE   IP              NODE      NOMINATED NODE   READINESS GATES
grafana-6ccd56f4b6-72458                1/1     Running   7 (19d ago)    40d   10.10.235.180   worker1   <none>           <none>
istio-ingressgateway-57c665985b-bfxd9   1/1     Running   6 (45m ago)    36d   10.10.235.158   worker1   <none>           <none>
istiod-78cc776c5b-hhm2b                 1/1     Running   6 (19d ago)    36d   10.10.235.129   worker1   <none>           <none>
jaeger-5d44bc5c5d-h6r9c                 1/1     Running   7 (19d ago)    40d   10.10.235.162   worker1   <none>           <none>
kiali-79b86ff5bc-q6gxg                  1/1     Running   7 (19d ago)    40d   10.10.235.160   worker1   <none>           <none>
prometheus-64fd8ccd65-pzff8             2/2     Running   14 (19d ago)   40d   10.10.235.161   worker1   <none>           <none>
```

# 5. Access thru Browser
```
# vi /etc/hosts
192.168.33.220 animals.example.com

# curl animals.example.com/cat/index.html
cat
<pre>
          | \\\                 /__\
         |.\\\ \               //--.i
         i \\\\ \             ///// i
         i \\\\\ \  _______  //////:|
         i.\\\\\\ --|i | i|-- /////.|
         |:\\\\  |||:i i iiii|  /// i
         \ \     / i|i | |||i\      i
         /       :| || i i| i       i
         |       |:  i | | |i|      \
         i        | || | i  i|       i
        |      /     | : |     \\    i
        |     //  __  :.:: __   \    i
        |        / _\     / _\       i
        |       | # \      # \       i
        i  ___--| \_)      \_)--___  |
        i====    \__/     \__/   ====|
        i==                        ==|
        i-            _-_           -i
       /|             \ /            i
     //  \    _ _ ---  | -- _       /
    /    _\- -   _--   |  --_ --_  /
   /    -  \_ _-  _-   |  --  -_  /_
    /     _ -\_ -- -__/ \__- -_ _/_ -
 //      -   _---__  \___/ --__/_  -
           _-                    -_
/
</pre>
```
```
# curl animals.example.com/dog/index.html
dog
<pre>
          ____
       ,-'-,  `---._
_______(0} `, , ` , )
V           ; ` , ` (                            ,'~~~~~~`,
`.____,- '  (,  `  , )                          :`,-'""`. ";
  `-------._);  ,  ` `,                         \;:      )``:
         )  ) ; ` ,,  :                          ``      : ';
        (  (`;:  ; ` ;:\                                 ;;;,
        (:  )``;:;;)`'`'`--.    _____     ____       _,-';;`
        :`  )`;)`)`'   :    "~~~     ~~~~~    ~~~`--',.;;;'
        `--;~~~~~      `  ,  ", """,  "  "   "` ",, \ ;``
          ( ;         ,   `                ;      `; ;
          (; ; ;      `                   ,`       ` :
           (; /            ;   ;          ` ;     ; :
           ;(_; ;  :   ; ; `; ;` ; ; ,,,""";}     `;
           : `; `; `  :  `  `,,;,''''   );;`);     ;
           ;' :;   ; : ``'`'           (;` :( ; ,  ;
           |, `;; ,``                  `)`; `(; `  `;
           ;  ;;  ``:                   `).:` \;,   `.
        ,-'   ;`;;:;`                   ;;'`;;  `)   )
         ~~~,-`;`;,"                    ~~~~~  ,-'   ;
            """"""                             `"""""
</pre>
```

# 6. HTTPS Access
# 6-1. Create certfile
```
$ openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout example.com.key -out example.com.crt
$ openssl req -out animals.example.com.csr -newkey rsa:2048 -nodes -keyout animals.example.com.key -subj "/CN=animals.example.com/O=animals organization"
$ openssl x509 -req -sha256 -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 0 -in animals.example.com.csr -out animals.example.com.crt
```
# 6-2. Create Secret and Ingress-gateway
```
$ kubectl create -n istio-system secret tls animals-credential --key=animals.example.com.key --cert=animals.example.com.crt
$ kubectl apply -f ingress-gateway-https.yaml
```
# 6-3. Let's access thru HTTPS
```
$ curl https://animals.example.com/cat/index.html -k
cat
<pre>
          | \\\                 /__\
         |.\\\ \               //--.i
         i \\\\ \             ///// i
         i \\\\\ \  _______  //////:|
         i.\\\\\\ --|i | i|-- /////.|
         |:\\\\  |||:i i iiii|  /// i
         \ \     / i|i | |||i\      i
         /       :| || i i| i       i
         |       |:  i | | |i|      \
         i        | || | i  i|       i
        |      /     | : |     \\    i
        |     //  __  :.:: __   \    i
        |        / _\     / _\       i
        |       | # \      # \       i
        i  ___--| \_)      \_)--___  |
        i====    \__/     \__/   ====|
        i==                        ==|
        i-            _-_           -i
       /|             \ /            i
     //  \    _ _ ---  | -- _       /
    /    _\- -   _--   |  --_ --_  /
   /    -  \_ _-  _-   |  --  -_  /_
    /     _ -\_ -- -__/ \__- -_ _/_ -
 //      -   _---__  \___/ --__/_  -
           _-                    -_
/
</pre>
```
```
$ curl https://animals.example.com/dog/index.html -k
dog
<pre>
          ____
       ,-'-,  `---._
_______(0} `, , ` , )
V           ; ` , ` (                            ,'~~~~~~`,
`.____,- '  (,  `  , )                          :`,-'""`. ";
  `-------._);  ,  ` `,                         \;:      )``:
         )  ) ; ` ,,  :                          ``      : ';
        (  (`;:  ; ` ;:\                                 ;;;,
        (:  )``;:;;)`'`'`--.    _____     ____       _,-';;`
        :`  )`;)`)`'   :    "~~~     ~~~~~    ~~~`--',.;;;'
        `--;~~~~~      `  ,  ", """,  "  "   "` ",, \ ;``
          ( ;         ,   `                ;      `; ;
          (; ; ;      `                   ,`       ` :
           (; /            ;   ;          ` ;     ; :
           ;(_; ;  :   ; ; `; ;` ; ; ,,,""";}     `;
           : `; `; `  :  `  `,,;,''''   );;`);     ;
           ;' :;   ; : ``'`'           (;` :( ; ,  ;
           |, `;; ,``                  `)`; `(; `  `;
           ;  ;;  ``:                   `).:` \;,   `.
        ,-'   ;`;;:;`                   ;;'`;;  `)   )
         ~~~,-`;`;,"                    ~~~~~  ,-'   ;
            """"""                             `"""""
</pre>
```

# 7. JWT authentication

> https://istiobyexample.dev/jwt/ <br>
> https://tech.jxpress.net/entry/deploy-secure-api-with-istio-and-auth0-in-5-mins

# 7-1. Create JWT
> https://auth0.com/

Step1)
![ingress_auth0_1](https://github.com/developer-onizuka/Istio_ingressGateway/blob/main/ingress_auth0_1.png)

Step2)
![ingress_auth0_2](https://github.com/developer-onizuka/Istio_ingressGateway/blob/main/ingress_auth0_2.png)

Step3)
![ingress_auth0_1](https://github.com/developer-onizuka/Istio_ingressGateway/blob/main/ingress_auth0_3.png)

# 7-2. RequestAuthentication and AuthorizationPolicy
```
$ kubectl apply -f auth0-jwt.yaml
```

# 7-3. Access without JWT Auth
```
$ curl -k https://animals.example.com/cat/index.html
RBAC: access denied
```

# 7-4. Access without JWT Auth
```
$ curl -k https://animals.example.com/cat/index.html --header 'authorization: Bearer eyJhbGxxx.....xxx'
cat
<pre>
          | \\\                 /__\
         |.\\\ \               //--.i
         i \\\\ \             ///// i
         i \\\\\ \  _______  //////:|
         i.\\\\\\ --|i | i|-- /////.|
         |:\\\\  |||:i i iiii|  /// i
         \ \     / i|i | |||i\      i
         /       :| || i i| i       i
         |       |:  i | | |i|      \
         i        | || | i  i|       i
        |      /     | : |     \\    i
        |     //  __  :.:: __   \    i
        |        / _\     / _\       i
        |       | # \      # \       i
        i  ___--| \_)      \_)--___  |
        i====    \__/     \__/   ====|
        i==                        ==|
        i-            _-_           -i
       /|             \ /            i
     //  \    _ _ ---  | -- _       /
    /    _\- -   _--   |  --_ --_  /
   /    -  \_ _-  _-   |  --  -_  /_
    /     _ -\_ -- -__/ \__- -_ _/_ -
 //      -   _---__  \___/ --__/_  -
           _-                    -_
/
</pre>
```

You can use [Modheader](https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj) with google-chrome instead of cURL.
