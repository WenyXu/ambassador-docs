---
description: "A simple three step guide to installing $productName$ and quickly get started routing traffic from the edge of your Kubernetes cluster to your services."
---

import Alert from '@material-ui/lab/Alert';
import GettingStartedEdgeStack20Tabs from './gs-tabs'

# $productName$ quick start

<div class="docs-article-toc">
<h3>Contents</h3>

* [1. Installation](#1-installation)
* [2. Routing Traffic from the Edge](#2-routing-traffic-from-the-edge)
* [What's Next?](#img-classos-logo-srcimageslogopng-whats-next)

</div>

## 1. Installation

We'll start by installing $productName$ into your cluster.

**We recommend using Helm** but there are other options below to choose from.

<GettingStartedEdgeStack20Tabs/>

<Alert severity="success"><b>Success!</b> You have installed $productName$, now let's get some traffic flowing to your services.</Alert>

## 2. Routing traffic from the edge

Like any other Kubernetes object, Custom Resource Definitions (CRDs) are used to declaratively define $productName$’s desired state. The workflow you are going to build uses a simple demo app and the **AmbassadorMapping CRD**, which is the core resource that you will use with $productName$. It lets you route requests by host and URL path from the edge of your cluster to Kubernetes services.

1. First, create an AmbassadorListener resources:

```
kubectl apply -f - <<EOF
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: $productDeploymentName$-listener-8080
  namespace: $productNamespace$
spec:
  port: 8080
  protocol: HTTP
  securityModel: XFP
  hostBinding:
    namespace:
      from: ALL
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorListener
metadata:
  name: $productDeploymentName$-listener-8443
  namespace: $productNamespace$
spec:
  port: 8443
  protocol: HTTPS
  securityModel: XFP
  hostBinding:
    namespace:
      from: ALL
EOF
```

2. Apply the YAML for the “Quote of the Moment" service.

  ```
  kubectl apply -n $productNamespace$ -f https://app.getambassador.io/yaml/v2-docs/latest/quickstart/qotm.yaml
  ```

  <Alert severity="info">The Service and Deployment are created in the $productName$ namespace.  You can use <code>kubectl get services,deployments quote --namespace $productNamespace$ </code> to see their status.</Alert>

3. Copy the configuration below and save it to a file called `quote-backend.yaml` so that you can create an AmbassadorMapping on your cluster. This AmbassadorMapping tells $productName$ to route all traffic inbound to the `/backend/` path to the `quote` Service.

  ```yaml
  ---
  apiVersion: x.getambassador.io/v3alpha1
  kind: AmbassadorMapping
  metadata:
    name: quote-backend
    namespace: $productNamespace$
  spec:
    hostname: "*"
    prefix: /backend/
    service: quote
  ```

4. Apply the configuration to the cluster:

  ```
  kubectl apply -f quote-backend.yaml
  ```

  With our AmbassadorMapping created, now we need to access it!

5. Store the $productName$ load balancer IP address to a local environment variable. You will use this variable to test accessing your service.

  ```
  export LB_ENDPOINT=$(kubectl -n $productNamespace$ get svc  $productDeploymentName$ \
    -o "go-template={{range .status.loadBalancer.ingress}}{{or .ip .hostname}}{{end}}")
  ```

6. Test the configuration by accessing the service through the $productName$ load balancer:

  ```
  $ curl -Lki https://$LB_ENDPOINT/backend/

    HTTP/1.1 200 OK
    content-type: application/json
    date: Wed, 23 Jun 2021 16:49:46 GMT
    content-length: 163
    x-envoy-upstream-service-time: 0
    server: envoy

    {
        "server": "serene-grapefruit-gjd4yodo",
        "quote": "The last sentence you read is often sensible nonsense.",
        "time": "2021-06-23T16:49:46.613322198Z"
    }
  ```

<Alert severity="success"><b>Victory!</b> You have created your first $productName$ AmbassadorMapping, routing a request from your cluster's edge to a service!</Alert>

## <img class="os-logo" src="../../images/logo.png"/> What's next?

Explore some of the popular tutorials on $productName$:

* [Intro to Mappings](../../topics/using/intro-mappings/): declaratively routes traffic from
the edge of your cluster to a Kubernetes service
* [AmbassadorHost resource](../../topics/running/host-crd/): configure a hostname and TLS options for your ingress.
* [Rate Limiting](../../topics/using/rate-limits/rate-limits/): create policies to control sustained traffic loads

$productName$ has a comprehensive range of [features](/features/) to
support the requirements of any edge microservice.

To learn more about how $productName$ works, read the [$productName$ Story](../../about/why-ambassador).
