# Deploying-Cassandra-with-a-StatefulSet

<img width="1661" height="2178" alt="default" src="https://github.com/user-attachments/assets/c5c66edf-4f51-4ad0-969f-3bcf600a71ad" />



Cassandra Deployment on Kubernetes
Overview
This project aims to deploy a Cassandra database on a Kubernetes cluster using a set of resources to ensure stability, persistent storage, and secure access. The setup includes configurations to run three Cassandra nodes using a StatefulSet, with persistent storage provided by PersistentVolumeClaims and PersistentVolumes, and configuration/security settings managed via ConfigMap and Secret.
Main Components

StatefulSet (sts-cassandra):

Manages three Pods running Cassandra: cassandra-0, cassandra-1, cassandra-2.
Ensures ordered creation and termination of Pods to maintain data consistency.
Associates each Pod with a PersistentVolumeClaim for data storage.


Service (svc-cassandra):

Provides access to the Pods via the CQL port (9042/TCP).
Configured as a NodePort to allow external connections to the cluster.


PersistentVolumeClaim (pvc-cassandra-data):

Requests persistent storage for each Pod (10GB per PVC).
Links to a StorageClass (standard) and PersistentVolume.


PersistentVolume (pv-cassandra-data):

Provides the physical storage backing the PVCs.
Uses hostPath as an example (can be replaced with a cloud provider like AWS EBS).


StorageClass (standard):

Defines the storage type (e.g., no-provisioner or a cloud provider).
Available cluster-wide for all namespaces.


ConfigMap (cm-kube-root-ca.crt):

Contains a CA certificate (dummy example) for securing communications.
Mounted as a volume in the Pods.


Secret (sa-default):

Stores sensitive data (e.g., Cassandra password).
Mounted as a volume in the Pods.



Relationships Between Components

The StatefulSet (sts-cassandra) manages the Pods and links to the Service (svc-cassandra) via serviceName.
Each Pod uses a PVC (cassandra-data-<pod-name>), which binds to a PV through the StorageClass (standard).
ConfigMap and Secret provide configuration and security settings to the Pods via mounted volumes.
The Service enables internal and external access to the Pods on port 9042.

Prerequisites

A running Kubernetes cluster (version 1.18 or later).
kubectl installed and configured to access the cluster.
Cassandra container image available (uses cassandra:latest in the example).
Storage backend available (either hostPath for testing or a cloud provider like AWS EBS).

Installation Steps

Apply the StorageClass:
kubectl apply -f sc-standard.yaml

This creates a cluster-wide standard StorageClass for provisioning storage.

Apply the PersistentVolume (if using manual storage):
kubectl apply -f pv-cassandra-data.yaml

This defines a PersistentVolume for the PVCs (optional if using dynamic provisioning).

Apply the ConfigMap:
kubectl apply -f cm-kube-root-ca.crt.yaml

This provides the CA certificate for the Pods.

Apply the Secret:
kubectl apply -f sa-default.yaml

This stores sensitive data like the Cassandra password.

Apply the Service:
kubectl apply -f svc-cassandra.yaml

This creates a NodePort Service to access the Cassandra Pods.

Apply the StatefulSet:
kubectl apply -f sts-cassandra.yaml

This deploys the Cassandra Pods with persistent storage.

Verify the Deployment:

Check the Pods:kubectl get pods -n default


Check the PVCs:kubectl get pvc -n default


Check the Service:kubectl get svc -n default





Accessing Cassandra
Internal Access (within the cluster)

Use the Service DNS: svc-cassandra.default.svc.cluster.local:9042.
Connect to individual Pods using: cassandra-<n>.svc-cassandra.default.svc.cluster.local:9042 (e.g., cassandra-0.svc-cassandra.default.svc.cluster.local:9042).

External Access

The Service is exposed as a NodePort (e.g., port 30042).
Get the Node IP:kubectl get nodes -o wide


Connect using <node-ip>:30042 with a Cassandra client (e.g., cqlsh):cqlsh <node-ip> 30042 -u cassandra -p cassandra_password


Alternatively, use kubectl port-forward for local testing:kubectl port-forward service/svc-cassandra 9042:9042

Then connect to localhost:9042.

Example: Connecting with Python
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider

auth_provider = PlainTextAuthProvider(username='cassandra', password='cassandra_password')
cluster = Cluster(['<node-ip>'], port=30042, auth_provider=auth_provider)
session = cluster.connect('keyspace_name')

# Execute a query
rows = session.execute('SELECT * FROM table_name')
for row in rows:
    print(row)

cluster.shutdown()

Notes

StorageClass: The standard StorageClass is cluster-scoped and can be used by PVCs in any namespace. Replace no-provisioner with a cloud-specific provisioner (e.g., kubernetes.io/aws-ebs) for production.
ConfigMap: Contains a dummy CA certificate (ca.crt). Update it with your actual certificate for secure communication.
Secret: Stores the Cassandra password (cassandra_password). Ensure it matches your Cassandra configuration.
Customization:
Update cassandra.yaml via ConfigMap if needed (e.g., for cluster name or authentication settings).
Modify the Service type to LoadBalancer for cloud environments to get an external IP.


Security: Enable SSL/TLS in Cassandra and update the ConfigMap/Secret accordingly for production.
Scaling: To scale the StatefulSet, update replicas in sts-cassandra.yaml and apply the change.

Troubleshooting

Pods not starting: Check Pod logs (kubectl logs cassandra-0) and ensure the Cassandra image and storage are configured correctly.
Storage issues: Verify that PVCs are bound to PVs (kubectl describe pvc -n default).
Connection issues: Ensure the NodePort is accessible and firewall rules allow traffic on port 30042.

Files

sts-cassandra.yaml: Defines the StatefulSet for Cassandra Pods.
svc-cassandra.yaml: Defines the Service for accessing Pods.
pvc-cassandra-data.yaml: Example PVC (auto-generated by StatefulSet).
pv-cassandra-data.yaml: Defines the PersistentVolume.
sc-standard.yaml: Defines the cluster-wide StorageClass.
cm-kube-root-ca.crt.yaml: Contains the CA certificate.
sa-default.yaml: Stores sensitive data like passwords.

For further customization or support, refer to the Kubernetes and Cassandra documentation or contact the cluster administrator.
