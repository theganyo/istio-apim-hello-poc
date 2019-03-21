# demo

## install istio

    kubectl apply -f $ISTIO/install/kubernetes/istio-demo.yaml

    kubectl label namespace default istio-injection=enabled

## install workloads, virtualservices, gateway

    kubectl apply -f operator/helloworld.yaml
    kubectl apply -f operator/httpbin.yaml
    kubectl apply -f operator/virtualservice-helloworld.yaml
    kubectl apply -f operator/virtualservice-httpbin.yaml
    kubectl apply -f operator/gateway.yaml

## access virtualservices via gateway

    curl -i -H host:httpbin.com 0/headers
    curl -i -H host:helloworld.vs 0/hello

    curl -i -H host:api.helloworld.com 0/hello

## access api via gateway (fails)

    curl -i -H host:api.helloworld.com 0/hello

## apply authentication

    kubectl apply -f operator/authentication.yaml

## apply api gateway

    kubectl apply -f generated/virtualservice-api-helloworld-com.yaml
    kubectl apply -f filters/_claims-to-headers.yaml
    kubectl apply -f filters/api-classification.yaml
    kubectl apply -f filters/api-destination.yaml

    curl -i -H host:httpbin.com 0/headers

    curl -i -H host:helloworld.vs 0/hello

    curl -i -H host:api.helloworld.com 0/hello

## access via gateway

    SILVER=`apigee-istio -o theganyo1-eval -e test token create -i 8B0pYeAZbC30xB1MJDEHup9YDWOV6y9A -s NJSzhph9QCZGIHt0`
    apigee-istio -o theganyo1-eval -e test token inspect <<< ${SILVER}
  
    curl -i -H host:api.helloworld.com 0/hello -H "Authorization: Bearer ${SILVER}"

    GOLD=`apigee-istio -o theganyo1-eval -e test token create -i f9N8m6T4HGeOzojIKX9UrWsHbAW5pARG -s zqLlwIq9A6gXYfDp`
    apigee-istio -o theganyo1-eval -e test token inspect <<< ${GOLD}
  
    curl -i -H host:api.helloworld.com 0/hello -H "Authorization: Bearer ${GOLD}"

## policy

    kubectl apply -f generated/policy-inject-response.yaml

    curl -i -H host:api.helloworld.com 0/hello

## access inside of mesh

    kubectl apply -f $ISTIO/samples/sleep/sleep.yaml
  
    SLEEP=`kubectl get po -l app=sleep -o 'jsonpath={.items[0].metadata.name}'`

    kubectl exec -it ${SLEEP} -c sleep -- curl -i api.helloworld.com/hello -H "Authorization: Bearer ${SILVER}"
  
    kubectl exec -it ${SLEEP} -c sleep -- curl -i api.helloworld.com/hello -H "Authorization: Bearer ${GOLD}"
