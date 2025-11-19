### ALB Controller ingress manifest use examples

#### ALB ingress URL redirect method
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: default
  name: ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/actions.redirect-to-eks: >
      {"type":"redirect","redirectConfig":{"host":"<example.com>","path":"/eks/","port":"443","protocol":"HTTPS","query":"k=v","statusCode":"HTTP_302"}}
spec:
  ingressClassName: alb
  rules:
    - host: 
          http:
             paths:
                - path: /eks
                  pathType: Exact
                  backend:
                     service:
                        name: redirect-to-eks
                        port:
                           name: use-annotation
```

#### Forward action to redirect the host domain request to destination host
```
alb.ingress.kubernetes.io/actions.forward-single-tg: >
      {"type":"forward","targetGroupARN": "arn-of-your-target-group"}
spec:
  rules:
    - http:
        paths:
          - path: /path1
            backend:
              serviceName: forward-single-tg
              servicePort: use-annotation
```
- Note: Use the TG name, you must create the target group with an external IP address type; there register the EC2 host IP where the actual application was hosted while skipping all the top layers, like the external domain name and middle layer load balancer.
