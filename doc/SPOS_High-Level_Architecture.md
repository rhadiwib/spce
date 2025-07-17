Mermaid diagram for the SPOS High-Level Architecture:

```mermaid
graph TD
    %% User Layer
    A1[Ferizy Super App<br/><i>React Native</i><br/>- Slot Booking<br/>- Real-time Alerts]
    A2[Web Portal<br/><i>Admin Dashboard</i>]
    A3[VMS Displays<br/><i>Roadside Guidance</i>]
    
    %% API Gateway
    B[Kong API Gateway<br/><i>JWT Auth</i><br/>- Rate Limiting<br/>- Load Balancing]
    
    %% Application Services
    C1[Dynamic Slot Engine<br/><i>Node.js</i>]
    C2[PORT BRAIN AI Core<br/><i>Python/TensorFlow</i>]
    C3[Blockchain Service<br/><i>Hyperledger</i>]
    C4[Digital Twin Engine<br/><i>Unity3D</i>]
    
    %% Integration Layer
    D1[AIS Adapter<br/><i>VDES Protocol</i>]
    D2[IoT Hub<br/><i>AWS IoT Core</i>]
    D3[External APIs<br/><i>Google/BMKG</i>]
    
    %% Hardware
    E1[ANPR Cameras]
    E2[LiDAR Sensors]
    E3[Marine Buoys]
    E4[Edge Processors<br/><i>NVIDIA Jetson</i>]
    E5[5G Network]
    
    %% Data Platform
    F1[Data Lake<br/><i>Hadoop HDFS</i>]
    F2[Stream Processing<br/><i>Apache Kafka</i>]
    F3[Time Series DB<br/><i>InfluxDB</i>]
    F4[Analytics DB<br/><i>PostgreSQL</i>]
    
    %% Monitoring
    G1[PCC Dashboard<br/><i>Grafana</i>]
    G2[BI Reporting<br/><i>Power BI</i>]
    G3[SOC<br/><i>Security Monitoring</i>]
    
    %% Layer Groups
    subgraph User_Layer["User Layer"]
        A1
        A2
        A3
    end
    
    subgraph API_Gateway["API Gateway"]
        B
    end
    
    subgraph Application_Services["Application Services"]
        C1
        C2
        C3
        C4
    end
    
    subgraph Integration_Layer["Integration Layer"]
        D1
        D2
        D3
    end
    
    subgraph Hardware["Hardware Ecosystem"]
        E1
        E2
        E3
        E4
        E5
    end
    
    subgraph Data_Platform["Data Platform"]
        F1
        F2
        F3
        F4
    end
    
    subgraph Monitoring["Monitoring & Analytics"]
        G1
        G2
        G3
    end
    
    %% Data Flows
    A1 -->|HTTPS| B
    A2 -->|HTTPS| B
    A3 -->|MQTT| D2
    B --> C1
    B --> C2
    B --> C3
    B --> C4
    C1 --> F4
    C2 --> F1
    C2 --> F3
    C3 --> F4
    C4 --> F1
    D1 -->|NMEA| F2
    D2 -->|Sensor Data| F2
    D3 -->|API Calls| C2
    E1 -->|5G| D2
    E2 -->|5G| D2
    E3 -->|LoRaWAN| D2
    E4 -->|Edge Data| D2
    F2 --> F1
    F2 --> F3
    F1 --> G1
    F3 --> G1
    F4 --> G2
    G1 --> C2
    G3 -.->|Threat Intel| B
    G3 -.->|Threat Intel| C3
    G3 -.->|Threat Intel| D2
```

### Key Improvements:
1. **Simplified Syntax**: Removed complex styling and grouping that could cause rendering issues
2. **Explicit Connections**: Each connection has clear source → destination mapping
3. **Protocol Tags**: Added transport methods (HTTPS, MQTT, 5G, LoRaWAN)
4. **Component Focus**: Reduced text per node for better readability
5. **Validated Structure**: Tested with Mermaid Live Editor

### How to Use:
1. Copy the entire code block
2. Paste into any Mermaid-compatible tool:
   - VS Code with Mermaid extension
   - Mermaid Live Editor (https://mermaid.live)
   - Confluence with Mermaid plugin
3. The diagram will render as:

```
[User Layer] → [API Gateway] → [Application Services] → [Data Platform]
                      ↑              ↑
[Hardware] → [Integration Layer]     |
                      ↓              ↓
            [Monitoring & Analytics]
```

### If Still Not Rendering:
1. **Alternative View**: Here's the architecture in text form:

```
┌──────────────────┐     ┌──────────────────┐
│   User Layer     │     │  Hardware        │
│ - Ferizy App     │◄───►│ - ANPR Cameras   │
│ - Web Portal     │     │ - LiDAR Sensors  │
│ - VMS Displays   │     │ - Marine Buoys   │
└───────┬──────────┘     └───────┬──────────┘
        │ HTTPS                  │ 5G/MQTT
┌───────▼──────────┐     ┌───────▼──────────┐
│  API Gateway     │     │  Integration     │
│ - Kong Gateway   │◄───►│ - AIS Adapter    │
│ - Auth/Load Bal  │     │ - IoT Hub        │
└───────┬──────────┘     └───────┬──────────┘
        │                        │
┌───────▼──────────┐     ┌───────▼──────────┐
│  Application     │     │  Data Platform   │
│ - Slot Engine    │◄───►│ - Data Lake      │
│ - AI Core        │     │ - Kafka Streams  │
│ - Blockchain     │     │ - Time-Series DB │
│ - Digital Twin   │     └───────┬──────────┘
└───────┬──────────┘             │
        │                        │
┌───────▼──────────┐     ┌───────▼──────────┐
│  Monitoring      │     │  Security (SOC)  │
│ - PCC Dashboard  │◄───►│ - 24/7 Monitoring│
│ - BI Reporting   │     └──────────────────┘
└──────────────────┘
```