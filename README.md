# k8s
# **K8s Documentation**

**Audience:** This document details about the deploying and troubleshooting k8s objects for nnew users

# contents

- [Pod Deployment](#pod-deployment)
- [Contents](#contents)





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
