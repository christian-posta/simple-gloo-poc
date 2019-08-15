# Access logging use case

We can control [Enovy's access logging](https://www.envoyproxy.io/docs/envoy/latest/configuration/access_log) to print only the details in which we are interested including custom headers. 

Gloo configures access logging through the [Gloo Gateway](https://gloo.solo.io/introduction/concepts/#gateways). 

Please see [Gloo's access logging configuration](https://gloo.solo.io/gloo_routing/gateway_configuration/access_logging/).

## Prerequisites

Make sure you have the petstore application and the default virtual service deployed. You should be able to run the following command successfully:

```
curl -H "Host: default.test" $(glooctl proxy url)/petstore

[{"id":1,"name":"Dog","status":"available"},{"id":2,"name":"Cat","status":"pending"}]
```
## Enabling access logging

Now, let's update the gateway config with the following:

```yaml
apiVersion: gateway.solo.io.v2/v2
kind: Gateway
metadata:
  annotations:
    origin: default
  name: gateway
  namespace: gloo-system
spec:
  bindAddress: '::'
  bindPort: 8080
  gatewayProxyName: gateway-proxy-v2
  httpGateway: {}
  plugins:
    accessLoggingService:
      accessLog:
      - fileSink:
          path: /dev/stdout
          jsonFormat:
            startTime: "%START_TIME%"
            httpMethod: "%REQ(:METHOD)%"
            path: "%REQ(X-ENVOY-ORIGINAL-PATH?:PATH)%"
            protocol: "%PROTOCOL%"
            responseCode: "%RESPONSE_CODE%"
            duration: "%DURATION%"
            xff: "%REQ(X-FORWARDED-FOR)%" 
            userAgent: "%REQ(USER-AGENT)%"
            authority: "%REQ(:AUTHORITY)%"
            upstreamHost: "%UPSTREAM_HOST%"
            customHeader: "%REQ(custom-header-here)%"
  useProxyProto: false

```

For convenience, you can run the following:

```bash
kubectl apply -f gateway.yaml
gateway.gateway.solo.io.v2/gateway created
```

Now, watch the logs in the Gloo Gateway when you make a call to the service. 

Watching the logs:
```bash
kubectl logs -f $(kubectl get pod -l gloo=gateway-proxy-v2 -n gloo-system -o jsonpath='{.items[0].metadata.name}') -n gloo-system
```

Send a request in:

```bash
curl -H "Host: default.test" $(glooctl proxy url)/petstore
```

The watch for a log record like this:

```json
{"customHeader":"-","startTime":"2019-08-15T15:39:05.922Z","duration":"1","xff":"-","upstreamHost":"10.28.3.133:8080","httpMethod":"GET","responseCode":"200","authority":"default.test","path":"/petstore","protocol":"HTTP/1.1","userAgent":"curl/7.54.0"}
```