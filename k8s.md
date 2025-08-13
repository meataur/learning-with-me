```mermaid
flowchart TD
    %% Legend
    subgraph Legend [Legend]
        style Legend fill:#f8f9fa,stroke:#343a40,stroke-width:1px
        L1[🔴 Control Plane / Master Node]
        L2[🔵 Worker Nodes & Pods]
        L3[🟡 Services]
        L4[🟢 Ingress / LoadBalancer]
        L5[🟠 External Clients / DNS]
        L6[🟣 Observability / Storage]
    end

    %% External World
    Internet[🟠 Internet / External Users]
    DNS[🟠 DNS / Domain Name]
    style Internet fill:#fff3cd,stroke:#856404,stroke-width:2px
    style DNS fill:#fff3cd,stroke:#856404,stroke-width:2px

    %% Load Balancer
    subgraph LB [🟢 Cloud Load Balancer / External LB]
        style LB fill:#d4edda,stroke:#155724,stroke-width:2px
        LoadBalancer[LoadBalancer Service]
    end

    %% Ingress Layer
    subgraph IngressLayer [🟢 Ingress Controller]
        style IngressLayer fill:#c3e6cb,stroke:#155724,stroke-width:2px
        Ingress[Ingress Controller]
    end

    %% Control Plane
    subgraph CP [🔴 Control Plane / Master Node]
        style CP fill:#f8d7da,stroke:#d9534f,stroke-width:2px
        APIServer[API Server]
        Scheduler[Scheduler]
        ControllerManager[Controller Manager]
        ETCD[etcd<br/>Cluster State]
        APIServer -->|Stores Cluster State| ETCD
        Scheduler -->|Schedules Pods| APIServer
        ControllerManager -->|Manages Controllers| APIServer
    end

    %% Worker Nodes
    subgraph WorkerNodes [🔵 Worker Nodes]
        style WorkerNodes fill:#d1ecf1,stroke:#17a2b8,stroke-width:2px
        subgraph Node1 [Node 1]
            style Node1 fill:#cce5ff,stroke:#004085
            Kubelet1[Kubelet]
            KubeProxy1[Kube-Proxy]
            PodA[Pod A<br/>App1]
            PodB[Pod B<br/>App1]
            Kubelet1 -->|Receives instructions| APIServer
            KubeProxy1 --> PodA
            KubeProxy1 --> PodB
        end

        subgraph Node2 [Node 2]
            style Node2 fill:#cce5ff,stroke:#004085
            Kubelet2[Kubelet]
            KubeProxy2[Kube-Proxy]
            PodC[Pod C<br/>App2]
            PodD[Pod D<br/>App2]
            Kubelet2 -->|Receives instructions| APIServer
            KubeProxy2 --> PodC
            KubeProxy2 --> PodD
        end
    end

    %% Services
    subgraph ServicesLayer [🟡 Services]
        style ServicesLayer fill:#fff3cd,stroke:#856404,stroke-width:2px
        ClusterIP1[ClusterIP App1]
        ClusterIP2[ClusterIP App2]
        NodePort1[NodePort App1]
        NodePort2[NodePort App2]
    end

    %% Observability & Storage
    subgraph Observability [🟣 Monitoring / Logging / Storage]
        style Observability fill:#e2e3f3,stroke:#6f42c1,stroke-width:2px
        Prometheus[Prometheus<br/>Metrics Collection]
        Grafana[Grafana<br/>Dashboard]
        Fluentd[Fluentd / Logging]
        PV[Persistent Volumes]
        PVC[Persistent Volume Claims]
        StorageClass[Storage Class]
    end

    %% External connections with flow numbers
    Internet -.->|1. User Request| DNS
    DNS -.->|2. Resolves Domain| LoadBalancer
    LoadBalancer -.->|3. Routes Traffic| Ingress
    Ingress -.->|4. Routes to Service| ClusterIP1
    Ingress -.->|4. Routes to Service| ClusterIP2

    %% Service to Pod connections
    ClusterIP1 -->|5. Requests| PodA
    ClusterIP1 -->|5. Requests| PodB
    ClusterIP2 -->|5. Requests| PodC
    ClusterIP2 -->|5. Requests| PodD

    NodePort1 -->|Optional External Access| PodA
    NodePort1 -->|Optional External Access| PodB
    NodePort2 -->|Optional External Access| PodC
    NodePort2 -->|Optional External Access| PodD

    %% Pod-to-Pod communication
    PodA -. Internal API .->|6. Calls| PodC
    PodB -. Internal API .->|6. Calls| PodD

    %% Observability connections
    PodA -->|Metrics| Prometheus
    PodB -->|Metrics| Prometheus
    PodC -->|Metrics| Prometheus
    PodD -->|Metrics| Prometheus
    Prometheus -->|Dashboards| Grafana

    PodA -->|Logs| Fluentd
    PodB -->|Logs| Fluentd
    PodC -->|Logs| Fluentd
    PodD -->|Logs| Fluentd

    %% Storage connections
    PodA -->|Claims Storage| PVC
    PodB -->|Claims Storage| PVC
    PodC -->|Claims Storage| PVC
    PodD -->|Claims Storage| PVC
    PVC --> PV
    PV --> StorageClass

    %% Control Plane to Kubelets
    APIServer -->|7. Sends instructions| Kubelet1
    APIServer -->|7. Sends instructions| Kubelet2
```
