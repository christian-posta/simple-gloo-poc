# External cluster routing with static upstreams

In this use case, we specify routing rules to services that live outside of our cluster or are created using [static upstreams](https://gloo.solo.io/gloo_routing/virtual_services/routes/route_destinations/single_upstreams/static_upstream/). 

## Prerequisites

Let's make sure we create the upstream for the external service like this:

```bash
kubectl apply -f external-cluster-routing/jsonplaceholder-us.yaml 
```


## Testing external service

Now, let's create a new VirtualService config with the following:

```yaml
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: test-header
  namespace: gloo-system
spec:
  virtualHost:
    domains:
      - 'use.case.extrouting'
    routes:
      - matcher:
          prefix: /
        routeAction:
          single:
            upstream:
              name: json-upstream
              namespace: gloo-system
   
```

In this example, we are matching on a domain `use.case.extrouting` and a simple `/` predicate and routing to the `json-upstream` we created in the prerequisites. 



For convenience, you can run the following:

```bash
kubectl apply -f basic-verb-and-uri/basic-routing-vs.yaml 

```

To make a request that matches:

```bash
curl -v  -X GET -H "Host: use.case.basicrouting" http://35.230.74.185:80/petstore
```
