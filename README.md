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

# 4. Check Istio gateway created for each service
```
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
```

# 5. Access thru Browser
```
# curl 192.168.121.220/cat/index.html
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
# curl 192.168.121.220/dog/index.html
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
