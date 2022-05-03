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
If you find "HTTP/2 404" in curl's verbose output, then Check if another gateway was created. If you find it, delete it.
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

# Appendix. Configure a mutal TLS ingress gateway (HTTPS)

- mTLS authentication flow <br>
<img src="https://miro.medium.com/max/1190/1*AeygepIJxBwo9zbmgjGB2w.png" width="640">

# A-0. Create secrets for mutual TLS
```
$ kubectl delete -n istio-system secrets animals-credential
$ kubectl create -n istio-system secret generic animals-credential --from-file=tls.key=animals.example.com.key --from-file=tls.crt=animals.example.com.crt --from-file=ca.crt=example.com.crt
```
A TLS Secret with keys tls.key and tls.crt, as described above. For mutual TLS, a ca.crt key can be used in addition.
```
$ kubectl -n istio-system describe secrets animals-credential
Name:         animals-credential
Namespace:    istio-system
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
ca.crt:   1180 bytes
tls.crt:  1054 bytes
tls.key:  1704 bytes
```

# A-1. Change the gateway’s definition to set the TLS mode to MUTUAL
```
$ kubectl edit gateway animals-gateway
...
    tls:
      credentialName: animals-credential
      mode: MUTUAL
```

# A-2. Attempt to send an HTTPS request using the prior approach and see how it fails
```
$ curl -k https://animals.example.com/cat/index.html -v
*   Trying 192.168.33.220:443...
* TCP_NODELAY set
* Connected to animals.example.com (192.168.33.220) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Request CERT (13):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Certificate (11):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=animals.example.com; O=animals organization
*  start date: Mar 21 09:01:35 2022 GMT
*  expire date: Mar 21 09:01:35 2023 GMT
*  issuer: O=example Inc.; CN=example.com
*  SSL certificate verify result: unable to get local issuer certificate (20), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x55975a3fd3b0)
> GET /cat/index.html HTTP/2
> Host: animals.example.com
> user-agent: curl/7.68.0
> accept: */*
> 
* TLSv1.3 (IN), TLS alert, unknown (628):
* OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert certificate required, errno 0
* Failed receiving HTTP2 data
* OpenSSL SSL_write: SSL_ERROR_ZERO_RETURN, errno 0
* Failed sending HTTP2 data
* Connection #0 to host animals.example.com left intact
curl: (56) OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert certificate required, errno 0
```
# A-3. Generate client certificate and private key
```
$ openssl req -out client.example.com.csr -newkey rsa:2048 -nodes -keyout client.example.com.key -subj "/CN=client.example.com/O=client organization"
$ openssl x509 -req -sha256 -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 1 -in client.example.com.csr -out client.example.com.crt
```
# A-4. cURL
```
$ curl https://animals.example.com/cat/index.html --cert client.example.com.crt --key client.example.com.key -k
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
# A-5. Google Chrome
You should convert the client certificate + the private key into a PKCS12 certificate.
```
$ openssl pkcs12 -export -inkey client.example.com.key -in client.example.com.crt -out client.example.com.p12
```

You can import it as below:

<img src="https://github.com/developer-onizuka/Istio_ingressGateway/blob/main/mtls_clientCert1.png" width="640">
<br>
<img src="https://github.com/developer-onizuka/Istio_ingressGateway/blob/main/mtls_clientCert2.png" width="480">

# 7. JWT authentication

> https://istiobyexample.dev/jwt/ <br>
> https://tech.jxpress.net/entry/deploy-secure-api-with-istio-and-auth0-in-5-mins

# 7-1. Create JWT
> https://auth0.com/

**Step1) Create a new Identifier** <br>

<img src="https://github.com/developer-onizuka/Istio_ingressGateway/blob/main/ingress_auth0_1.png" width="480"><br>

The identifier is used to identify which service the Istio IngressGateway takes API caller to. The Istio IngressGateway might host 3 or more services for API callers. API publisher(means you) should manage the JWT indivisually for each service.<br>
<img src="https://github.com/developer-onizuka/Diagrams/blob/main/istioAuth0/Auth0.drawio.png">


**Step2) Retrieve token to the API (Bearer token)**<br>
<img src="https://github.com/developer-onizuka/Istio_ingressGateway/blob/main/ingress_auth0_2.png" width="640">

# 7-2. Create RequestAuthentication and AuthorizationPolicy
```
$ kubectl apply -f auth0-jwt.yaml
```
Please note issuer and jwksUri in auth0-jwt.yaml should be replaced with yours.
```
  - issuer: https://dev-dnkam18n.us.auth0.com/
    jwksUri: https://dev-dnkam18n.us.auth0.com/.well-known/jwks.json
```

# 7-3. Start Auth0 aware Pod
```
$ kubectl delete -f cat/nginx_cat.yaml
$ kubectl apply -f cat/nginx_cat_jwt.yaml
```

# 7-4. Access without Bearer token
```
$ curl -k https://animals.example.com/cat/index.html
RBAC: access denied
```

# 7-5. Access with Bearer token
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

<img src="https://github.com/developer-onizuka/Istio_ingressGateway/blob/main/ingress_auth0_3.png" width="480">
