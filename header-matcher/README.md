# Header present based routing

In this use case, we'll only match requests where the "Authorization" is present and not null.

## Prerequisites

Make sure you have the petstore application and the default virtual service deployed. You should be able to run the following command successfully:

```
curl -H "Host: default.test" $(glooctl proxy url)/petstore

[{"id":1,"name":"Dog","status":"available"},{"id":2,"name":"Cat","status":"pending"}]
```


## Testing external service

Now, let's create a new VirtualService config with the following:

```
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: header-present-vs
  namespace: gloo-system
spec:
  virtualHost:
    domains:
      - 'use.case.headermatch'
    routes:
      - matcher:
          exact: /petstore
          headers:
            - name: authorization
        routeAction:
          single:
            upstream:
              name: default-petstore-8080
              namespace: gloo-system
        routePlugins:
          prefixRewrite:
            prefixRewrite: /api/pets    
      - matcher:
          prefix: /
        directResponseAction:
          status: 405
          body: '<Payload contentType="application/json">{"Error" : "{fault.name}", "ErrorCode" : "405", "ErrorDesc" : "Method Not Allowed" }</Payload>'      
   
```

In this example, we are matching on a domain `use.case.headermatch` and a predicate that requires the `authorization` header. Note that it's specified as an all-lowercase header as Envoy automatically converts everything to HTTP/2 internally, and lower-case headers is a strict requirement for HTTP/2.



For convenience, you can run the following:

```bash
kubectl apply -f header-matcher/header-matching-vs.yaml  -n gloo-system

```

To make a request that matches:

```bash
curl -v -H "authorization: foo"  -X GET -H "Host: use.case.headermatch" http://35.230.74.185:80/petstore
```
