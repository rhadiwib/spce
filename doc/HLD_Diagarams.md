```mermaid
graph TD
    %% User Layer
    U[Users: Passengers, Port Operators, ASDP, Police, Navy]

    %% Hardware Ecosystem Layer
    subgraph Hardware Ecosystem Data Acquisition and Actuation
        H1[ANPR Cameras: ≥98% accuracy]
        H2[LiDAR Docking Sensors: ±2cm accuracy]
        H3[IoT Sensors: Vibration,Marine_Buoys,Traffic,Weather]
        H4[VMS Variable Message Signs]
        H5[Automated Marshalling LEDs]
    end

    %% Edge Computing Layer
    subgraph Edge Computing in Local Processing
        EC[AWS IoT Greengrass / Azure IoT Edge / NVIDIA Jetson AGX Orin]
    end

    %% Network Layer (Connectivity Backbone)
    subgraph Network Layer
        N1[5G Private Network: <10ms latency]
        N2[VSAT: Offshore Connectivity]
        N3[Fiber Optic: Core Data Links]
        N4[LoRaWAN: Long-range IoT]
    end

    %% Data Layer (Ingestion & Storage)
    subgraph Data Layer
        D1[Apache Kafka: Real-time Streaming 500K msg/sec]
        D2[Unified Data Lake: Hadoop HDFS]
        D3[InfluxDB: Time-series Database]
        D4[Apache Spark ML: Data Cleansing 15-30% AIS noise]
    end

    %% Analytics Layer (AI & Simulation)
    subgraph Analytics Layer
        A1[PORT BRAIN AI Core: TensorFlow/PyTorch ML Models]
        A2[Digital Twin: Siemens Tecnomatix / Unity 3D]
    end

    %% Application Layer (User-Facing & Orchestration)
    subgraph Application Layer
        AP1[Ferizy Super App: React Native MobileApps]
        AP2[Ferizy Backend: Node.js/Python Microservices]
        AP3[Port Command Center Dashboard: Grafana/Kibana]
        AP4[API Gateway: Kong / AWS API Gateway]
        AP5[Blockchain Ticketing: Hyperledger Fabric]
    end

    %% Security Layer (Cross-Cutting Protection)
    subgraph Security Layer
        S1[Zero Trust Architecture]
        S2[AES-256 Encryption]
        S3[Biometric Authentication]
        S4[Quantum-Resistant Encryption]
    end

    %% Human Layer (Training & Adoption)
    subgraph Human Layer
        HM1[AR/VR Simulators: Microsoft HoloLens]
        HM2[Digital Literacy Programs: BPSDM Partnership]
    end

    %% Flow of Data and Interaction
    U -- Interact --> AP1
    AP1 <--> AP2
    AP2 <--> AP4
    AP2 -- Via AP4 --> D1
    H1 -- Data --> EC
    H2 -- Data --> EC
    H3 -- Data --> EC
    EC -- Processed Data --> N1
    N1 --> D1
    N2 --> D1
    N4 --> D1

    D1 --> D2
    D1 --> D3
    D2 -- Cleaned by --> D4
    D3 -- Cleaned by --> D4
    D4 --> A1
    D4 --> A2

    AP4 -- Immutable Records --> D2

    AP2 -- Requests --> AP4
    AP2 -- Via AP4 --> D1
    AP2 -- Via AP4 --> D2
    AP2 -- Via AP4 --> D3

    AP5 -- API Calls --> AP2
    AP5 -- API Calls --> A1
    AP5 -- API Calls --> AP3

    A1 -- Insights, Commands --> AP3
    A1 -- Orchestrates --> H4
    A1 -- Orchestrates --> H5
    A2 -- Simulations --> A1
    AP3 -- Displays --> U
    U -- Input --> AP3

    %% Cross-Layer Interactions and Dependencies
    N1 --- Real_time_Sync --- A1
    N1 --- Edge_Connectivity --- EC
    N2 --- Offshore_Data --- A1
    N3 --- High_Bandwidth_Core --- D2
    N3 --- High_Bandwidth_Core --- A1
    N3 --- High_Bandwidth_Core --- AP2

    S1 --- Protects_all_layers_D_A_AP
    S2 --- Encrypts_data_in_D_AP
    S3 --- Authenticates_U_to_AP_A

    HM1 --- Training --> U
    HM2 --- Education --> U
```

---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---


---

```mermaid
```

---

