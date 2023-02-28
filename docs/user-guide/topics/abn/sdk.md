---
template: main.html
---

# Iter8 SDK

Iter8 provides an SDK for application developers to facilitate A/B/n testing of [apps](../../../glossary.md#app). The SDK provides three [gRPC](https://grpc.io/) APIs, namely, `GetRoute`, `SetRoute`, and `WriteMetric`. [Iter8 service](service.md) provides the server-side implementation of these APIs. The client bindings for these APIs can be generated and used within a client written in [any gRPC supported language](https://grpc.io/docs/languages/).

## GetRoute

`GetRoute` takes as input [a subject ID](../appspec.md#app-id) and a user ID, and returns a [variant number](../../../glossary.md#version-number). Its protobuf definition is as follows.

```proto
// GetRoute returns a variant number for the given RequestMeta (subject/user ID combination)
rpc GetRoute(RequestMeta) returns(Variant) {}

message RequestMeta {
  // subject ID
  string subject = 1;
  // user ID
  string user = 2;
}

message Variant {
  // variant number
  int32 number = 1;
}
```

The [Iter8 service](service.md) uses [Iter8 storage](../../../glossary.md#iter8-storage) to record every route returned by `GetRoute`.

### Illustration

A typical usage scenario for the `GetRoute` API is illustrated below. In this scenario, the application has a client-server architecture. The backend server is the subject that is being A/B tested and has two variants. Whenever the client needs to send a request to the backend, it calls `GetRoute` to determine the backend variant, and routes the request appropriately.


![Backend A/B/n testing](../../../tutorials/abn/images/abn.backend.png)

### Guarantees

`GetRoute` provides the following three guarantees.

1. **Readiness**: The variant returned by `GetRoute` is guaranteed to be [ready](../../../glossary.md#readiness). This enables safe and error-free updates and deletion of Kubernetes app resources. Assuming at least one variant is ready, `GetRoute` returns the `OK` status. Otherwise, it returns an error status.

2. **Weighted routing**: Given a set of variants that are ready, and a user, `GetRoute` maps the user to a variant in this set with probability proportional to the [variant weight](../../../glossary.md#weight).

3. **User stickiness**: Given a set of variants that are ready, and a user, distinct `GetRoute` calls for the user will return the same variant. User stickiness is essential during A/B/n testing for correct attribution of metrics to variants and consistent end-user experience during the experiment.

## SetRoute
`SetRoute` takes as input a subject ID, a user ID, and a variant number, and records this route in [Iter8 storage](../../../glossary.md#iter8-storage). Its protobuf definition is as follows.

```proto
// Record a route in Iter8 storage
rpc SetRoute(Route) returns (google.protobuf.Empty) {}

message Route {
  // subject ID
  string subject = 1;
  // user ID
  string user = 2;
  // variant number
  int32 number = 3;
}
```

### Illustration

Two usage scenarios for the `SetRoute` API are illustrated below.

=== "SetRoute called by client"
    The distributed Kubernetes application consists of a client and backend server. The backend is the subject that is being A/B tested and has two variants. The requests from the client to the backend are routed through a service mesh, proxy, or load balancer. Whenever the client sends a request to the backend, it calls `SetRoute` to record the route.

    ![Downstream call](images/abn.backend.cr.png){: style="width:90%"}

=== "SetRoute called by the subject"
    Requests are routed to different variants of the subject through a service mesh, proxy, or load balancer. Whenever a variant receives a request, it calls `SetRoute` to record the route.

    ![App call](images/abn.app.cr.png){: style="width:75%"}

## WriteMetric
`WriteMetric` takes as input a subject ID, a user ID, a [counter metric](../../../glossary.md#counter) name, and a metric value, and records the metric value in [Iter8 storage](../../../glossary.md#iter8-storage). Its protobuf definition is as follows.

```proto
// Record a counter metric value in Iter8 storage
rpc WriteMetric(Counter) returns (google.protobuf.Empty) {}

message Counter {
  // metric name
  string metric = 1;
  // metric value
  double value = 2;
  // subject ID
  string subject = 3;
  // user ID
  string user = 4;
}
```

The `WriteMetric` API is illustrated as part of the [GetRoute](#illustration) and [SetRoute](#illustration_1) scenarios. 

## Usage examples

The client bindings for the Iter8 SDK APIs can be generated, embedded, and invoked within a client written in [any gRPC supported language](https://grpc.io/docs/languages/). The code used in the [A/B testing tutorial](../../../tutorials/abn/quickstart.md) is in [this repo](https://github.com/iter8-tools/docs): you can refer to [client implementations](https://github.com/iter8-tools/docs/tree/main/samples/abn-sample/frontend) in [`Node`](https://github.com/iter8-tools/docs/tree/main/samples/abn-sample/frontend/node), [`Python`](https://github.com/iter8-tools/docs/tree/main/samples/abn-sample/frontend/python), and [`Go`](https://github.com/iter8-tools/docs/tree/main/samples/abn-sample/frontend/go). We now provide an overview of the main steps involved in client development. 

### Generate client bindings
Our first step is to generate the client bindings from the [abn.proto](https://github.com/iter8-tools/iter8/blob/master/abn/grpc/abn.proto) file. The client bindings provide the glue code needed to interact with the [Iter8 service](service.md).

=== "Node"
    The [gRPC Node.js library](https://grpc.io/docs/languages/node/basics/#loading-service-descriptors-from-proto-files) dynamically generates service descriptors and client bindings from `.proto` files loaded at runtime. So, there is nothing to do in this step.

=== "Python"
    Generate Python client bindings from [abn.proto](https://github.com/iter8-tools/iter8/blob/master/abn/grpc/abn.proto) as described [here](https://grpc.io/docs/languages/python/basics/#generating-client-and-server-code).

=== "Go"
    Generate Go client bindings from [abn.proto](https://github.com/iter8-tools/iter8/blob/master/abn/grpc/abn.proto) as described [here](https://grpc.io/docs/languages/go/basics/#generating-client-and-server-code). Place the generated files as part of a `go` package.

### Import client bindings
Import the code generated above in your client.

=== "Node"
    ```javascript
    # use the correct import path
    var PROTO_PATH = '/the/path/to/abn.proto';
    var grpc = require('@grpc/grpc-js');
    var protoLoader = require('@grpc/proto-loader');
    // Suggested options for similarity to existing grpc.load behavior
    var packageDefinition = protoLoader.loadSync(
        PROTO_PATH,
        {keepCase: true,
        longs: String,
        enums: String,
        defaults: true,
        oneofs: true
        });
    var protoDescriptor = grpc.loadPackageDefinition(packageDefinition);
    // The protoDescriptor object has the full package hierarchy
    var abn = protoDescriptor.abn;
    ```

=== "Python"
    ```python
    import grpc
    # use the correct import path
    import abn_pb2
    import abn_pb2_grpc   
    ```

=== "Go"
    ```go
    import (
        "google.golang.org/grpc"
        "google.golang.org/grpc/credentials/insecure"
        # use the correct import path
        pb "the/package/containing/the/go/client/bindings"
    )
    ```

### Create stub
To call the Iter8 service methods, we first need to create a *stub*. Suppose you have [installed the Iter8 service](../../../getting-started/install.md#install-or-upgrade-iter8-service) in the `iter8-system` namespace. You can then create the client stub as follows.

=== "Node"
    ```javascript
    var stub = new abn.ABN(
        'iter8-service.iter8-system.svc.cluster.local:50051',
         grpc.credentials.createInsecure());
    ```

=== "Python"
    ```python
    channel = grpc.insecure_channel('iter8-service.iter8-system.svc.cluster.local:50051')
    stub = abn_pb2_grpc.ABNStub(channel) 
    ```

=== "Go"
    ```go
    var opts []grpc.DialOption
    ...
    conn, err := grpc.Dial('iter8-service.iter8-system.svc.cluster.local:50051', opts...)
    if err != nil {
    ...
    }
    defer conn.Close()
    ```

!!! warning "The following section are WIP ... ignore for now"

### Invoke GetRoute
### Invoke SetRoute
### Invoke WriteMetric

<!-- ### Invoke GetRoute

!!! warning "This section is yet to be fixed ... ignore for now"

=== "Node"
    ```javascript
    const trackToRoute = {
        "backend":   "http://backend.default.svc.cluster.local:8091",
        "backend-candidate-1": "http://backend-candidate-1.default.svc.cluster.local:8091",
    }

    var application = new messages.Application();
    application.setName('default/backend');
    application.setUser(req.header('X-User'));
    client.lookup(application, function(err, session) {
        if (err || (session.getTrack() == '')) {
            // use default route (see above)
            console.warn("error or null")
        } else {
            // use route determined by recommended track
            console.info('lookup suggested track %s', session.getTrack())
            route = trackToRoute[session.getTrack()];
        }

        // call backend service using route
        ...
    });
    ```

=== "Python"

=== "Go"
    ```go
    trackToRoute = map[string]string{
        "backend":             "http://backend.default.svc.cluster.local:8091",
        "backend-candidate-1": "http://backend-candidate-1.default.svc.cluster.local:8091",
    }

    route := trackToRoute["backend"]
    user := req.Header["X-User"][0]
    s, err := (*client).Lookup(
        ctx,
        &pb.Application{
            Name: "default/backend",
            User: user,
        },
    )
    if err == nil && s != nil {
        r, ok := trackToRoute[s.GetTrack()]
        if ok {
            route = r
        }
    }

    // call backend service using route
    ...
    ```

### Invoke SetRoute

???+ warning "This section is yet to be fixed ... ignore for now"

=== "Node"
    ```javascript
    var stub = new abn.ABNClient(
        'iter8-service.iter8-system.svc.cluster.local:50051',
         grpc.credentials.createInsecure());
    ```

=== "Python"
    ```python
    channel = grpc.insecure_channel('iter8-service.iter8-system.svc.cluster.local:50051')
    stub = route_guide_pb2_grpc.RouteGuideStub(channel) 
    ```

=== "Go"
    ```go
    var opts []grpc.DialOption
    ...
    conn, err := grpc.Dial('iter8-service.iter8-system.svc.cluster.local:50051', opts...)
    if err != nil {
    ...
    }
    defer conn.Close()
    ```

### Invoke WriteMetric

???+ warning "This section is yet to be fixed ... ignore for now"

As an example, a single metric named *sample_metric* is assigned a random value between 0 and 100 and written.

=== "Node"
    ```javascript
    var mv = new messages.MetricValue();
    mv.setName('sample_metric');
    mv.setValue(random({min: 0, max: 100, integer: true}).toString());
    mv.setApplication('default/backend');
    mv.setUser(user);
    ```

=== "Python"

=== "Go"
    ```go
    _, _ = (*client).WriteMetric(
        ctx,
        &pb.MetricValue{
            Name:        "sample_metric",
            Value:       fmt.Sprintf("%f", rand.Float64()*100.0),
            Application: "default/backend",
            User:        user,
        },
    )
    ``` -->
