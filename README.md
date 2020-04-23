# k8s
# **K8s Documentation**

**Audience:** This document details about the deploying and troubleshooting k8s objects for nnew users

# contents

- [Pod Deployment](#pod-deployment)
- [Contents](#contents)
- [Deployment with rolling update and roll back](#deployment-with-rolling-update-and-roll-back)
- [Stateffullset Deployment](#stateffullset-deployment)






### Pod Deployment

To deploy Pod use available manifest file from manifest folder. (**Note** Please change the pull secret where the image is getting downloaded )

```
oc create -f http-echo-pod.yml -n namespace
```

To Verify that pod is accesible with in cluster.Go to any other pod terminal and run the following command with the pod IP

```
/ $ curl -v 10.128.247.126:8080
*   Trying 10.128.247.126:8080...
* TCP_NODELAY set
* Connected to 10.128.247.126 (10.128.247.126) port 8080 (#0)
> GET / HTTP/1.1
> Host: 10.128.247.126:8080
> User-Agent: curl/7.66.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Thu, 23 Apr 2020 01:23:58 GMT
< Content-Length: 276
< Content-Type: text/plain; charset=utf-8
<
Hello, world!
Protocol: HTTP/1.1!
Hostname: test-http-echo1
Welcome, you are connected to node: worker0.dev.k8s.ford.com.
Running on Pod: test-http-echo1.
In namespace: test1234.
With IP address: 10.128.247.126.
Running as service account: default.
* Connection #0 to host 10.128.247.126 left intact
```
```
ping 10.128.247.126

```


[Back to Top](#contents)


### Deployment with rolling update and roll back

To deploy the deployment run the following command

```
oc create -f manifests/http-echo-deploy.yml -n namespace

```
Deploy service manifest to create service for the above deployment

```
oc create -f manifests/http-echo-svc.yml -n namespace

```

To test this pod is accesible vid podIP, ServiceIP and Service Hostname

```
#Curl Pod IP from another Pod Terminal
/ $ curl -v 10.128.247.126:8080
*   Trying 10.128.247.126:8080...
* TCP_NODELAY set
* Connected to 10.128.247.126 (10.128.247.126) port 8080 (#0)
> GET / HTTP/1.1
> Host: 10.128.247.126:8080
> User-Agent: curl/7.66.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Thu, 23 Apr 2020 01:23:58 GMT
< Content-Length: 276
< Content-Type: text/plain; charset=utf-8
<
Hello, world!
Protocol: HTTP/1.1!
Hostname: test-http-echo1
Welcome, you are connected to node: worker0.dev.k8s.ford.com.
Running on Pod: test-http-echo1.
In namespace: test1234.
With IP address: 10.128.247.126.
Running as service account: default.
* Connection #0 to host 10.128.247.126 left intact
```
```
ping 10.128.247.126

```

```
#Curl Service IP from another Pod Terminal
/ $ curl -v 172.30.202.36:80
*   Trying 172.30.202.36:80...
* TCP_NODELAY set
* Connected to 172.30.202.36 (172.30.202.36) port 80 (#0)
> GET / HTTP/1.1
> Host: 172.30.202.36
> User-Agent: curl/7.66.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Thu, 23 Apr 2020 01:39:58 GMT
< Content-Length: 309
< Content-Type: text/plain; charset=utf-8
<
Hello, world!
Protocol: HTTP/1.1!
Hostname: http-echo-deploy-5688ffc6f-6tt59
Welcome, you are connected to node: worker0.dev.k8s.ford.com.
Running on Pod: http-echo-deploy-5688ffc6f-6tt59.
In namespace: test1234.
With IP address: 10.128.247.65.
Running as service account: default.
* Connection #0 to host 172.30.202.36 left intact
Client: 10.129.37.171:51474/ $
```
```
#Curl Service hostname from another Pod Terminal
Client: 10.129.37.171:51474/ $ curl -v http-echo-deploy.test1234.svc
*   Trying 172.30.202.36:80...
* TCP_NODELAY set
* Connected to http-echo-deploy.test1234.svc (172.30.202.36) port 80 (#0)
> GET / HTTP/1.1
> Host: http-echo-deploy.test1234.svc
> User-Agent: curl/7.66.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Thu, 23 Apr 2020 01:40:36 GMT
< Content-Length: 309
< Content-Type: text/plain; charset=utf-8
<
Hello, world!
Protocol: HTTP/1.1!
Hostname: http-echo-deploy-5688ffc6f-6tt59
Welcome, you are connected to node: worker0.dev.k8s.ford.com.
Running on Pod: http-echo-deploy-5688ffc6f-6tt59.
In namespace: test1234.
With IP address: 10.128.247.65.
Running as service account: default.
* Connection #0 to host http-echo-deploy.test1234.svc left intact
Client: 10.129.37.171:52116/ $
```
```
# Curl PodIP with container Port and Service IP with service Port from local machine should give valid output
```
We can create route object to provide URL based connection

```
oc create -f http-echo-route.yml -n namespace

```
### Rolling Update

To Perform Rolling update increase our no of replicas to 2 and make suree changes are recorded

 ```
 oc scale deploy http-echo-deploy --replicas=2 -n test1234 --record
 
 **Note**: --record to record our deployment update change histoy
 ```
 To see the histor
 
 ```
 oc rollout history deployment/http-echo-deploy -n test1234
 ```
 
 Now we can do changes in the deployment by updating newer version of image to how rolling update is happening
 

```
oc set image deployment/http-echo-deploy http-echo=files.caas.ford.com/http-echo/http-echo:1.0.1 --record -n test1234
```
Verify the history to see the change

```

MGC12VCELXHTD6:retest rmanik20$ oc rollout history deployment/http-echo-deploy -n test1234
deployments "http-echo-deploy"
REVISION  CHANGE-CAUSE
1         oc scale deploy http-echo-deploy --replicas=3 --namespace=test1234 --record=true
2         oc set image deployment/http-echo-deploy http-echo=files.caas.ford.com/http-echo/http-echo:1.0.1 --record=true --namespace=test1234

```

Now we can do roll back to previous version

```
oc rollout undo deployment/http-echo-deploy --to-revision=1 -n test1234
```


### Statefullset Deployment

To Deploy statefullset run the following command

```
  oc create -f http-echo-sts.yml -n test1234
  
**Note** Make sure you update the required storage class name in the stateffullset yaml
```
Create stateffullset service

```
oc create -f http-echo-hless-svc.yml -n test1234
```

We can troubleshoot the pod and service as like we did it for Deployment. But note one thing Stateffullset should have headless service without clusterIP assigned



To test this pod is accesible vid podIP, ServiceIP and Service Hostname

```
#Curl Pod IP from another Pod Terminal
/ $ curl -v 10.128.247.126:8080
*   Trying 10.128.247.126:8080...
* TCP_NODELAY set
* Connected to 10.128.247.126 (10.128.247.126) port 8080 (#0)
> GET / HTTP/1.1
> Host: 10.128.247.126:8080
> User-Agent: curl/7.66.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Thu, 23 Apr 2020 01:23:58 GMT
< Content-Length: 276
< Content-Type: text/plain; charset=utf-8
<
Hello, world!
Protocol: HTTP/1.1!
Hostname: test-http-echo1
Welcome, you are connected to node: worker0.dev.k8s.ford.com.
Running on Pod: test-http-echo1.
In namespace: test1234.
With IP address: 10.128.247.126.
Running as service account: default.
* Connection #0 to host 10.128.247.126 left intact
```
```
ping 10.128.247.126

```

```
#Curl Service IP from another Pod Terminal
/ $ curl -v 172.30.202.36:80
*   Trying 172.30.202.36:80...
* TCP_NODELAY set
* Connected to 172.30.202.36 (172.30.202.36) port 80 (#0)
> GET / HTTP/1.1
> Host: 172.30.202.36
> User-Agent: curl/7.66.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Thu, 23 Apr 2020 01:39:58 GMT
< Content-Length: 309
< Content-Type: text/plain; charset=utf-8
<
Hello, world!
Protocol: HTTP/1.1!
Hostname: http-echo-deploy-5688ffc6f-6tt59
Welcome, you are connected to node: worker0.dev.k8s.ford.com.
Running on Pod: http-echo-deploy-5688ffc6f-6tt59.
In namespace: test1234.
With IP address: 10.128.247.65.
Running as service account: default.
* Connection #0 to host 172.30.202.36 left intact
Client: 10.129.37.171:51474/ $
```
```
#Curl Service hostname from another Pod Terminal
Client: 10.129.37.171:51474/ $ curl -v http-echo-deploy.test1234.svc
*   Trying 172.30.202.36:80...
* TCP_NODELAY set
* Connected to http-echo-deploy.test1234.svc (172.30.202.36) port 80 (#0)
> GET / HTTP/1.1
> Host: http-echo-deploy.test1234.svc
> User-Agent: curl/7.66.0
> Accept: */*
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Date: Thu, 23 Apr 2020 01:40:36 GMT
< Content-Length: 309
< Content-Type: text/plain; charset=utf-8
<
Hello, world!
Protocol: HTTP/1.1!
Hostname: http-echo-deploy-5688ffc6f-6tt59
Welcome, you are connected to node: worker0.dev.k8s.ford.com.
Running on Pod: http-echo-deploy-5688ffc6f-6tt59.
In namespace: test1234.
With IP address: 10.128.247.65.
Running as service account: default.
* Connection #0 to host http-echo-deploy.test1234.svc left intact
Client: 10.129.37.171:52116/ $
```
```
# Curl PodIP with container Port and Service IP with service Port from local machine should give valid output
```

We can create route object to provide URL based connection

```
oc create -f http-echo-route.yml -n namespace

```



[Back to Top](#contents)
