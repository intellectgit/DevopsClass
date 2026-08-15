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

Sealed Secrets
==========================
Sealed Secrets is a Kubernetes tool and custom controller created by Bitnami that allows you to safely manage and store sensitive data (like database passwords, API keys, or TLS certificates) inside your Git repositories.

Standard Kubernetes Secret manifests are stored in Git in plain text (only base64-encoded, which is easily decoded by anyone). Sealed Secrets solves this problem through encryption.

How Sealed Secrets Works
Client-Side Encryption: You use a command-line tool (kubeseal) to encrypt your regular Kubernetes Secret YAML into a SealedSecret custom resource on your local machine.

Safe to Commit: The resulting SealedSecret file contains encrypted ciphertext. It is completely safe to commit and push this file into a public or private Git repository.

Cluster-Side Decryption: When you apply the SealedSecret manifest to your EKS or Kubernetes cluster, a controller running inside the cluster (the Sealed Secrets Controller) uses a private key stored securely within the cluster to automatically decrypt it and generate the actual, usable Kubernetes Secret
