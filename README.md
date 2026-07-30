# vprofile-k8s: Deploying Multi-Tier VProfile Application on Kubernetes

This repository contains Kubernetes manifest files to deploy a multi-tier web application (**VProfile**) onto a Kubernetes cluster.

---

## 📐 System Architecture

The VProfile application consists of the following core components:

```
                      +-----------------------------+
                      |   NGINX Ingress Controller  |
                      +--------------+--------------+
                                     |
                                     v
                          +--------------------+
                          |   vproapp-service  |
                          +----------+---------+
                                     |
                                     v
                           +------------------+
                           |  vproapp (Pods)  |
                           +--+------------+--+
                              |            |
             +----------------+            +----------------+
             |                                              |
             v                                              v
  +--------------------+                         +--------------------+
  | vprocache01 (Svc)  |                         |    vprodb (Svc)    |
  +---------+----------+                         +---------+----------+
            |                                              |
            v                                              v
  +--------------------+                         +--------------------+
  |  vpromc (Memcached)|                         |   vprodb (MySQL)   |
  +--------------------+                         +---------+----------+
                                                           |
                                                           v
                                                 +--------------------+
                                                 | db-pv-claim (PVC)  |
                                                 +--------------------+

                       +--------------------+
                       |   vpromq01 (Svc)   |
                       +---------+----------+
                                 |
                                 v
                       +--------------------+
                       |vpromq01 (RabbitMQ) |
                       +--------------------+
```

- **Frontend / Application Layer**: `vproapp` (Java / Tomcat)
  - Uses *Init Containers* to verify the availability of Database and Caching services before initializing the main application container.
- **Database Layer**: `vprodb` (MySQL)
  - Uses a `PersistentVolumeClaim` (`db-pv-claim`) for persistent data storage.
- **Caching Layer**: `vpromc` (Memcached)
  - Provides in-memory caching service for application data.
- **Message Broker Layer**: `vpromq01` (RabbitMQ)
  - Manages asynchronous message queues.
- **Ingress Layer**: `vpro-ingress` (NGINX Ingress)
  - Routes external traffic to the internal `vproapp-service` via host domain `vprofile.hkhinfoteck.xyz`.
- **Secret Management**: `app-secret`
  - Manages sensitive credentials (MySQL root password, RabbitMQ password) centrally.

---

## 📁 Repository Structure

```text
.
├── README.md
└── kubedefs/
    ├── secret.yaml       # Kubernetes Secret storing DB & RabbitMQ passwords
    ├── dbpvc.yaml        # PersistentVolumeClaim for MySQL Database
    ├── dbdeploy.yaml     # Deployment manifest for MySQL Database
    ├── dbservice.yaml    # Service (ClusterIP) for MySQL Database
    ├── mcdeploy.yaml     # Deployment manifest for Memcached
    ├── mcservice.yaml    # Service (ClusterIP) for Memcached (vprocache01)
    ├── rmqdeploy.yaml    # Deployment manifest for RabbitMQ
    ├── rmqservice.yaml   # Service (ClusterIP) for RabbitMQ (vpromq01)
    ├── appdeploy.yaml    # Deployment manifest for VProfile Web App (with Init Containers)
    ├── appservice.yaml   # Service (ClusterIP) for Web App (vproapp-service)
    └── appingress.yaml   # Ingress routing configuration for Web App
```

