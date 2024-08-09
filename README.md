# K8s Gateway API Using Traefik

The purpose of this example is to provide instructions for running the K8s Gatewey API using Traefik.

## Software Requirements

- Helm v3.15.2 or newer

- Minikube v1.33.1 or newer

- OrbStack v1.6.4 or newer

Note: This tutorial was updated on macOS 14.6.1. The below steps doesn't work with Docker Desktop v4.31.1
because it doesn't expose Linux VM IP addresses to the host OS (i.e. macOS).

## Tutorial Installation

1.  clone github repository

    ```zsh
    git clone https://github.com/conradwt/k8s-gateway-api-using-traefik.git
    ```

2.  change directory

    ```zsh
    cd k8s-gateway-api-using-traefik
    ```

3.  create Minikube cluster

    ```zsh
    minikube start -p gateway-api-traefik
    ```

4.  install MetalLB

    ```zsh
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
    ```

5.  locate the K8s cluster's subnet

    ```zsh
    docker network inspect gateway-api-traefik | jq '.[0].IPAM.Config[0]["Subnet"]'
    ```

    The results should look something like the following:

    ```json
    "194.1.2.0/24",
    ```

    Then one can use an IP address range like the following:

    ```
    194.1.2.100-194.1.2.110
    ```

6.  create the `01-metallb-address-pool.yaml` file

    ```zsh
    cp 01-metallb-address-pool.yaml.example 01-metallb-address-pool.yaml
    ```

7.  update the `01-metallb-address-pool.yaml`

    ```yaml
    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
      name: demo-pool
      namespace: metallb-system
    spec:
      addresses:
        - 194.1.2.100-194.1.2.110
    ```

    Note: The IP range needs to be in the same range as the K8s cluster, `gateway-api-traefik`.

8.  apply the address pool manifest

    ```zsh
    kubectl apply -f 01-metallb-address-pool.yaml
    ```

9.  apply Layer 2 advertisement manifest

    ```zsh
    kubectl apply -f 02-metallb-advertise.yaml
    ```

10. apply deployment manifest

    ```zsh
    kubectl apply -f 03-nginx-deployment.yaml
    ```

11. apply service manifest

    ```zsh
    kubectl apply -f 04-nginx-service-loadbalancer.yaml
    ```

12. check that your service has an IP address

    ```zsh
    kubectl get svc nginx-service
    ```

    The results should look something like the following:

    ```text
    NAME            TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
    nginx-service   LoadBalancer   10.106.207.172   194.1.2.100   80:32000/TCP   17h
    ```

13. test connectivity to `nginx-service` endpoint via external IP address

    ```zsh
    curl 194.1.2.100
    ```

    The results should look something like the following:

    ```text
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    html { color-scheme: light dark; }
    body { width: 35em; margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif; }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    ```

14. install the Gateway API CRDs

    ```zsh
    kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/experimental-install.yaml
    ```

15. install/update Traefik RBAC

    ```zsh
    kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v3.1/docs/content/reference/dynamic-configuration/kubernetes-gateway-rbac.yml
    ```

16. create the Gateway and GatewayClass resources

    ```zsh
    helm repo add traefik https://traefik.github.io/charts
    helm repo update
    kubectl create namespace traefik
    helm upgrade --install --namespace traefik traefik traefik/traefik -f values.yaml
    ```

17. populate $PROXY_IP for future commands:

    ```zsh
    export PROXY_IP=$(kubectl get svc --namespace traefik traefik -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo $PROXY_IP
    ```

18. verify the proxy IP

    ```zsh
    curl -i $PROXY_IP
    ```

    The results should look something like the following:

    ```text
    HTTP/1.1 404 Not Found
    Content-Type: application/json; charset=utf-8
    Connection: keep-alive
    Content-Length: 48
    X-Kong-Response-Latency: 0
    Server: kong/3.0.0

    {"message":"no Route matched with those values"}
    ```

19. deploy the X service

    # TODO rewrite for our defined service.

    ```zsh
    kubectl create namespace whoami
    kubectl apply -f 06-sample-service.yaml
    ```

20. create HTTPRoute for our deployed service

    # TODO rewrite for our defined service.

    ```zsh
    kubectl apply -f 07-sample-httproute.yaml
    ```

21. test the routing rule

    # TODO rewrite for our defined service.

    ```zsh
    curl $PROXY_IP
    ```

    The results should look like this:

    ```text
    Hostname: whoami-98d7579fb-qq2sj
    IP: 127.0.0.1
    IP: ::1
    IP: 10.244.2.3
    IP: fe80::1060:7eff:fe82:5954
    RemoteAddr: 10.244.2.2:39066
    GET / HTTP/1.1
    Host: 194.1.2.101
    User-Agent: curl/8.9.1
    Accept: */*
    Accept-Encoding: gzip
    X-Forwarded-For: 194.1.2.3
    X-Forwarded-Host: 194.1.2.101
    X-Forwarded-Port: 80
    X-Forwarded-Proto: http
    X-Forwarded-Server: traefik-7c7587b647-vdmgt
    X-Real-Ip: 194.1.2.3
    ```

## References

- https://traefik.io/blog/getting-started-with-kubernetes-gateway-api-and-traefik
