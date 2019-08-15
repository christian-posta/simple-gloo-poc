# Direct response without proxying

In this use case, we set up matches for certain requests, and opt to NOT proxy them and immediately return a direct response (in this case an error message)

## Prerequisites

Make sure you have the petstore application and the default virtual service deployed. You should be able to run the following command successfully:

```
curl -H "Host: default.test" $(glooctl proxy url)/petstore

[{"id":1,"name":"Dog","status":"available"},{"id":2,"name":"Cat","status":"pending"}]
```

## Testing matching

Now, let's create a new VirtualService config with the following:

```yaml

apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: test-direct-response
  namespace: gloo-system
spec:
  virtualHost:
    domains:
      - 'use.case.directresponse'
    routes:
      - matcher:
          methods:
            - POST
          exact: /refer_a_friend/shorturls
        routeAction:
          single:
            upstream:
              name: default-petstore-8080
              namespace: gloo-system
        routePlugins:
          prefixRewrite:
            prefixRewrite: /api/pets    
      - matcher:
          methods:
            - GET
          regex: /campaigns/[0-9]+/products/[0-9]+/eligibility/.*
        routeAction:
          single:
            upstream:
              name: default-petstore-8080
              namespace: gloo-system
        routePlugins:
          prefixRewrite:
            prefixRewrite: /api/pets        
      - matcher:
          methods:
            - DELETE
            - POST
            - OPTIONS
            - PUT
          regex: /campaigns/[0-9]+/products/[0-9]+/eligibility/.*
        directResponseAction:
          status: 405
          body: '<Payload contentType="application/json">{"Error" : "{fault.name}", "ErrorCode" : "405", "ErrorDesc" : "Method Not Allowed" }</Payload>'                                 
      - matcher:
          prefix: /refer_a_friend/shorturls
        directResponseAction:
          status: 405
          body: '<Payload contentType="application/json">{"Error" : "{fault.name}", "ErrorCode" : "405", "ErrorDesc" : "Method Not Allowed" }</Payload>'
      - matcher:
          prefix: /
        directResponseAction:
          status: 405
          body: '<Payload contentType="application/json">{"Error" : "{fault.name}", "ErrorCode" : "405", "ErrorDesc" : "Method Not Allowed" }</Payload>'          
   
```

In this example, we are matching on a domain `use.case.directresponse` and various other predicates. 
In Gloo, the ordering of the matching rules are significant. The most restrictive matching predicates should be first while the most general should be last. In this example, we see for the most general matches, we will return errors (ie, `405`).


For convenience, you can run the following:

```bash
kubectl apply -f direct-response-vs.yaml
virtualservice.gateway.solo.io/test-direct-response created

```

To make a request that matches:

```bash
curl -v -X DELETE -H "Host: use.case.directresponse" http://35.230.74.185:80/campaigns/1/products/4/eligibility/foo
curl -v  -H "Host: use.case.directresponse" http://35.230.74.185:80/refer_a_friend/shorturls
```
