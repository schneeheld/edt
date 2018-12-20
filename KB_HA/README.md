
When you start a second Jenkins instance, you will observe the below logs in `jenkins.log`
on the first, already running, instance

```
Dec 18, 2018 4:20:47 PM com.cloudbees.jenkins.ha.singleton.HASingleton$3 viewAccepted

INFO: Cluster membership has changed to: [blackbird-44907|3] (2) [blackbird-44907, blackbird-57107]
Dec 18, 2018 4:20:47 PM com.cloudbees.jenkins.ha.singleton.HASingleton$3 viewAccepted

INFO: New primary node is JenkinsClusterMemberIdentity[member=blackbird-44907,weight=0,min=0]
Dec 18, 2018 4:20:47 PM com.cloudbees.jenkins.ha.singleton.HASingleton reactToPrimarySwitch

INFO: Elected as the primary node
Dec 18, 2018 4:20:47 PM com.cloudbees.jenkins.ha.singleton.HASingleton$3 viewAccepted

INFO: Cluster membership has changed to: [blackbird-44907|4] (1) [blackbird-44907]
Dec 18, 2018 4:20:47 PM com.cloudbees.jenkins.ha.singleton.HASingleton$3 viewAccepted

INFO: New primary node is JenkinsClusterMemberIdentity[member=blackbird-44907,weight=0,min=0]
Dec 18, 2018 4:20:47 PM com.cloudbees.jenkins.ha.singleton.HASingleton reactToPrimarySwitch

INFO: Elected as the primary node
```

### HTTP health-ckeck

The Load Balancer needs to performance a health check on the Jenkins Instances on `/ha/health-check`.

```
curl -i http://blackbird:8000/ha/health-check

HTTP/1.1 200 OK
Date: Wed, 19 Dec 2018 00:44:58 GMT
X-Content-Type-Options: nosniff
Content-Type: text/plain;charset=iso-8859-1
Transfer-Encoding: chunked
Server: Jetty(9.4.z-SNAPSHOT)

Running as primary
```

**Extended output**

On the primary instance, `curl -Lv http://blackbird:8001/ha/health-check` returns

```
curl -Lv http://blackbird:8000/ha/health-check
*   Trying 127.0.0.1...
* Connected to blackbird (127.0.0.1) port 8000 (#0)
> GET /ha/health-check HTTP/1.1
> Host: blackbird:8000
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Wed, 19 Dec 2018 00:25:46 GMT
< X-Content-Type-Options: nosniff
< Content-Type: text/plain;charset=iso-8859-1
< Transfer-Encoding: chunked
< Server: Jetty(9.4.z-SNAPSHOT)
< 
Running as primary
* Connection #0 to host blackbird left intact
```

On the standby instance, `curl -Lv http://blackbird:8001/ha/health-check` returns

```
*   Trying 127.0.0.1...
* Connected to blackbird (127.0.0.1) port 8001 (#0)
> GET /ha/health-check HTTP/1.1
> Host: blackbird:8001
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 503 Service Unavailable
< Date: Wed, 19 Dec 2018 00:25:51 GMT
< X-Content-Type-Options: nosniff
< Content-Type: text/plain;charset=iso-8859-1
< Transfer-Encoding: chunked
< Server: Jetty(9.4.z-SNAPSHOT)
< 
Running as standby
* Connection #0 to host blackbird left intact
```


## Nginx configuration
The specified URI is appended to the server domain name or IP address set for the server in the upstream block.
For the first server in the sample backend group, a health check requests the URI http://backend1/ha/health-check.

```
location / {
    proxy_pass http://backend;
    health_check uri=/some/path;
}
```

Specifying the Requested URI
https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/#specifying-the-requested-uri


