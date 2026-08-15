In general ingress controller itself act as API gateway .
 some API gateways - Kong gateway , AWS API gateway , nginx ingress controller 

 terminationGracePeriodSeconds: 10
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:	
            - labelSelector:	
                matchLabels:	
                  app: backend
              topologyKey: topology.kubernetes.io/zone

topologyKey: topology.kubernetes.io/zone
What it does: Defines the "domain" or boundary across which the anti-affinity rule is enforced. 
In this case, it looks at cloud provider Availability Zones (e.g., us-east-1a, us-east-1b).

Real-World Scenario: Your cloud provider (like AWS EKS) has multiple availability zones. 
By setting the topology key to zone, you ensure that your backend pods are distributed across different data centers/zones. 
If an entire AWS data center goes offline due to a power outage, your other backend pods running safely in a separate availability zone keep your application online with zero downtime.
