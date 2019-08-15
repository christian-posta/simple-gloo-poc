# Matching requests on verb and path

In this use case, we take a look at matching requests based on both context path AND HTTP method.

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
  name: basic-matching-vs
  namespace: gloo-system
spec:
  virtualHost:
    domains:
      - 'use.case.basicrouting'
    routes:
      - matcher:
          exact: /petstore
          methods:
          - GET
        routeAction:
          single:
            upstream:
              name: default-petstore-8080
              namespace: gloo-system
        routePlugins:
          prefixRewrite:
            prefixRewrite: /api/pets      
```

In this example, we are matching on a domain `use.case.basicrouting` and for context path `/petstore` and HTTP Method `GET`. When this request is matched, it will be sent to the `default-petstore-8080` upstream and in this case rewrite the path to `/api/pets`.


For convenience, you can run the following:

```bash
kubectl apply -f basic-routing-vs.yaml
virtualservice.gateway.solo.io/basic-matching-vs created

```

To make a request that matches:

```bash
curl -v  -X GET -H "Host: use.case.basicrouting" $(glooctl proxy url)/petstore
```

If you try any other METHOD or context path, you should get a `404`