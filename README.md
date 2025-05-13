# MongoDB Deployment in Kubernetes

## SIT737 - Cloud Native Application Development (Task 9.1P)

This repository contains the configuration files and documentation for deploying a MongoDB replica set in a Kubernetes cluster, as part of the SIT737 Cloud Native Application Development course at Deakin University.

## Overview

This project demonstrates how to deploy and configure a MongoDB database in a Kubernetes environment using StatefulSets, which provide stable network identities and persistent storage. The implementation follows cloud-native best practices, including:

- Configuration externalization via ConfigMaps
- Secure credential management with Kubernetes Secrets
- Data persistence with PersistentVolumes
- High availability with replica sets
- Health monitoring with probes

## Repository Structure

- `createStorageClass.yaml`: Defines how persistent volumes are dynamically provisioned
- `createConfigMap.yaml`: Stores non-sensitive MongoDB configuration
- `createHeadlessService.yaml`: Provides network identity to StatefulSet pods
- `createMongoDbSecret.yaml`: Securely stores MongoDB credentials
- `createPersistentVolumeClaim.yaml`: Requests storage from the Storage Class
- `createStatefulSet.yaml`: Defines the MongoDB replica set deployment

## Deployment Instructions

Follow these steps to deploy the MongoDB replica set in your Kubernetes cluster:

1. Create the Storage Class:
   ```bash
   kubectl apply -f createStorageClass.yaml
   ```

2. Create the ConfigMap:
   ```bash
   kubectl apply -f createConfigMap.yaml
   ```

3. Create the Secret:
   ```bash
   kubectl apply -f createMongoDbSecret.yaml
   ```

4. Create the StatefulSet:
   ```bash
   kubectl apply -f createStatefulSet.yaml
   ```

5. Create the Headless Service:
   ```bash
   kubectl apply -f createHeadlessService.yaml
   ```

6. Verify that the pods are running:
   ```bash
   kubectl get pods
   ```

## MongoDB Replica Set Configuration

After deploying the MongoDB instances, you need to initialize the replica set:

1. Access the primary node:
   ```bash
   kubectl exec -it mongo-0 -- /bin/sh
   ```

2. Launch the MongoDB shell:
   ```bash
   mongo
   ```

3. Initialize the replica set:
   ```javascript
   rs.initiate({
     _id: "rs0",
     members: [
       { _id: 0, host: "mongo-0.mongo.default.svc.cluster.local:27017" },
       { _id: 1, host: "mongo-1.mongo.default.svc.cluster.local:27017" },
       { _id: 2, host: "mongo-2.mongo.default.svc.cluster.local:27017" }
     ]
   })
   ```

4. Test CRUD operations:
   ```javascript
   use test
   db.createCollection("db1")
   
   // Create operation
   db.db1.insert({"name": "Krish", "course": "SIT737", "task": "9.1P"})
   
   // Read operation
   db.db1.find()
   
   // Update operation
   db.db1.updateOne({"name": "Krish"}, {$set: {"grade": "HD"}})
   
   // Delete operation
   db.db1.deleteOne({"task": "NotNeeded"})
   ```

## Verification of Replication

To verify that replication is working:

1. Access a secondary node:
   ```bash
   kubectl exec -it mongo-1 -- /bin/sh
   mongo
   ```

2. Enable reads on the secondary:
   ```javascript
   rs.slaveOk()
   ```

3. Verify that data is replicated:
   ```javascript
   db.db1.find()
   ```

## Connecting to the MongoDB Replica Set

Applications can connect to the MongoDB replica set using the following connection string:

```
mongodb://admin1:password123@mongo:27017/admin?replicaSet=rs0
```

Replace `admin1` and `password123` with the actual username and password from your ConfigMap and Secret.

## Features

- **High Availability**: Multiple MongoDB instances in a replica set
- **Data Persistence**: Storage that survives pod restarts
- **Secure Configuration**: Sensitive data stored in Kubernetes Secrets
- **Externalized Configuration**: Non-sensitive configuration stored in ConfigMaps
- **Health Monitoring**: Continuous verification of MongoDB instance health

## Requirements

- Kubernetes cluster (tested on version 1.20+)
- kubectl command-line tool
- Basic understanding of MongoDB and Kubernetes concepts

## References

- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [MongoDB Documentation](https://docs.mongodb.com/manual/)
- [StatefulSets in Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [MongoDB Replica Sets](https://docs.mongodb.com/manual/replication/)

## Author

Krish - SIT737 - Deakin University
