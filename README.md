# SOAP and XSLT Webinar

This README assumed that you have Gloo Edge Enterprise 1.8 installed. If not, please refer to:

* https://docs.solo.io/gloo-edge/latest/installation/enterprise/
* https://lp.solo.io/request-trial

## Demo #1: XSLT Official Guide

https://docs.solo.io/gloo-edge/latest/guides/traffic_management/request_processing/transformations/xslt_transformation/

```
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: world-cities-soap-service
spec:
  selector:
    matchLabels:
      app: world-cities-soap-service
  replicas: 1
  template:
    metadata:
      labels:
        app: world-cities-soap-service
    spec:
      containers:
        - name: world-cities-soap-service
          image: quay.io/solo-io/world-cities-soap-service:0.0.1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
EOF
```

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: world-cities-soap-service
  labels:
    app: world-cities-soap-service
spec:
  ports:
  - port: 8080
    protocol: TCP
  selector:
    app: world-cities-soap-service
EOF
```

```
kubectl apply -f - <<EOF
apiVersion: gateway.solo.io/v1
kind: VirtualService
metadata:
  name: default
  namespace: gloo-system
spec:
  virtualHost:
    domains:
      - '*'
    routes:
      - matchers:
        - prefix: /
        routeAction:
          single:
            upstream:
              # Upstream generated by gloo edge discovery
              name: default-world-cities-soap-service-8080
              namespace: gloo-system
        options:
          autoHostRewrite: true
EOF
```

```
curl $(glooctl proxy url) -H "SOAPAction:findCity" -H "content-type:application/xml" \
-d '<?xml version="1.0" encoding="UTF-8"?>
<Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/" xmlns:soap="http://schemas.xmlsoap.org/soap/">
   <Header />
   <Body>
      <Query>
         <CityQuery>south bo</CityQuery>
      </Query>
      \
   </Body>
</Envelope>' 
```

```
kubectl apply -f world-cities/virtualservice.yaml
```

```
curl $(glooctl proxy url) -d '{"cityQuery": "south bo"}' -H "SOAPAction:findCity" -H "content-type:application/json" | jq
```


## Demo #2: SOAP/REST Canary Deployment

### Create the Pet Store namespace.

```
kubectl create namespace petstore
```

### Deploy the SOAP Petstore Service

```
kubectl apply -n petstore -f deployment-soap.yaml
```

### Deploy the REST Petstore Service

```
kubectl apply -n petstore -f deployment-rest.yaml
```


### Apply the rate limiting:

```
kubectl apply -f rate-limiting.yaml
```

### Apply the VirtualService:

```
kubectl apply -f virtualservice.yaml
```

