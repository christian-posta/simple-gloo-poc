To get started with this POC, install a sample service:

```
kubectl apply -f petstore.yaml
```


Next, we create a virtual service in Gloo that can route to the petstore API. The Virtual Service looks like this:

```yaml
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: default
spec:
  virtualHost:
    domains:
    - 'default.test'
    routes:
    - matcher:
        exact: /petstore
      routeAction:
        single:
          upstream:
            name: default-petstore-8080
            namespace: gloo-system
      routePlugins:
        prefixRewrite:
          prefixRewrite: /api/pets
```


Let's apply the Virtual Service:

```
kubectl apply -f default-vs.yaml  -n gloo-system
```

Once this is created, let's try call the API 

```
curl -H "Host: default.test" $(glooctl proxy url)/petstore

[{"id":1,"name":"Dog","status":"available"},{"id":2,"name":"Cat","status":"pending"}]
```

The use cases we have in this POC are here:

* [Specifying access logging](./access-logging-usecase)
* [External cluster routing](./external-cluster-routing)
* [Header matching example](./header-matcher)
* [Direct response with error codes](./direct-response-error-codes)
* [Verb and URI matching](./basic-verb-and-uri)