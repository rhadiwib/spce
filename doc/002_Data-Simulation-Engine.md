You are a senior software architect and data engineer tasked with designing a **synthetic data simulation engine** for the Smart Port Command Center (SPCC) dashboard. Your deliverables are:  

1. **Simulation Engine Design**:  
   - Build a **production-grade simulation engine** that generates synthetic data mimicking:  
     - **AIS Data**: Vessel positions, speed, and ETA (Automatic Identification System behavior).  
     - **Ferizy Data**: Passenger/vehicle booking flows, traffic insights (Google Maps/Waze API patterns).  
     - **IoT Sensor Data**: Crane vibrations, weather conditions, and berth occupancy (MQTT/CoAP protocols).  
   - Ensure the engine replicates **real-world behavior** (e.g., noise patterns in AIS data, peak traffic seasonality in Ferizy).  

2. **Technical Requirements**:  
   - **Data Schema**: Define schemas for each data source (e.g., AIS JSON schema with `mmsi`, `latitude`, `speed`, `timestamp`).  
   - **Protocols**: Implement MQTT/CoAP for IoT sensors and REST/gRPC for Ferizy/AIS data streams.  
   - **Latency**: Simulate sub-5s latency for real-time updates (as required by SPCC [[001_Core_Components_and_Technical_Requirements_for_the_Dashboard.md]].  
   - **Scalability**: Support 3x peak season traffic (e.g., Idul Fitri) [[002_GMN_GoalsDigitalTransformation.md]].  

3. **Dashboard Integration**:  
   - Provide a **reference implementation** of the SPCC dashboard using:  
     - **React 18+** with TypeScript 5.x and Vite 5.x (per `React-Js_PoC_Dashboard_todo.md`).  
     - **Recharts/Leaflet** for vessel tracking and equipment health monitoring.  
     - **WebSocket** for real-time alerts (e.g., crane failures, congestion triggers).  
   - Ensure the dashboard consumes data from the simulation engine seamlessly.  

4. **Validation & Extensibility**:  
   - Include **validation checks** to verify synthetic data accuracy (e.g., 92% congestion forecasting alignment with SPCC goals [[001_Dashboard_Requirements_Deep_Analysis_.md]].  
   - Design the engine to allow **future integration with real data** (e.g., AIS/VDES APIs [[004_SPCC_Insight_Doc.md]].  

---

### **CONTEXT**
1. **Purpose**: Enable a proof-of-concept (PoC) for the SPCC dashboard without access to real-time AIS, Ferizy, or IoT sensor data.  
2. **Target Workflow**:  
   - Simulate **data pipelines** for SPCC components (PORT BRAIN Beta, IoT Data Platform).  
   - Validate SPCC’s predictive analytics (Apache Spark ML) and alert systems [[003_Core_Components_and_Technical_Requirements_for_the_Dashboard.md]].  
3. **Key Constraints**:  
   - Must adhere to **ISO 11064** (control room ergonomics) and **IEEE 2413-2019** (IoT interoperability).  
   - Support **Zero Trust Architecture** (AES-256 encryption for synthetic data streams) [[005_GRK_GoalsDigitalTransformation.md]].  

---

### **REQUIREMENTS**

#### **1. Simulation Engine**
- **Data Sources**:  
  - **AIS Simulation**:  
    - Generate vessel positions with realistic movement patterns (e.g., docking, queuing).  
    - Include noise/missing values (15–30% as per SPCC requirements [[001_Core_Components_and_Technical_Requirements_for_the_Dashboard.md]].  
  - **Ferizy Simulation**:  
    - Mock booking flows (vehicle/pedestrian counts) and dynamic pricing adjustments.  
    - Use Google Maps/Waze API patterns for traffic insights [[002_GMN_GoalsDigitalTransformation.md]].  
  - **IoT Sensor Simulation**:  
    - Replicate MQTT/CoAP protocols for constrained devices (e.g., crane vibration sensors).  
    - Include edge AI preprocessing (AWS IoT Greengrass patterns) for sub-5s latency [[000_GoalsDigitalTransformation.md]].  

- **Schema & Protocol Details**:  
  - **AIS Schema Example**:  
    ```json  
    {  
      "mmsi": "string",  
      "latitude": "number",  
      "longitude": "number",  
      "speed": "number",  
      "timestamp": "datetime"  
    }  
    ```  
  - **IoT Sensor Schema**:  
    ```json  
    {  
      "sensor_id": "string",  
      "value": "number",  
      "unit": "string",  
      "timestamp": "datetime"  
    }  
    ```  

#### **2. Dashboard Implementation**
- **Tech Stack**:  
  - **Frontend**: React 18+, TypeScript 5.x, Vite 5.x, Material-UI, Recharts.  
  - **Backend**: Node.js WebSocket server for real-time updates, Apache Kafka for data streaming [[004_SPCC_Insight_Doc.md]].  
- **Code Examples**:  
  - **WebSocket Integration**:  
    ```tsx  
    const useWebSocket = (url: string) => {  
      const [data, setData] = useState<any>(null);  
      useEffect(() => {  
        const ws = new WebSocket(url);  
        ws.onmessage = (event) => setData(JSON.parse(event.data));  
        return () => ws.close();  
      }, [url]);  
      return data;  
    };  
    ```  
  - **Predictive Analytics**:  
    - Use Apache Spark ML pipelines for berth allocation optimization [[000_GoalsDigitalTransformation.md]].  

#### **3. Validation & Roadmap**
- **Validation Steps**:  
  - Compare synthetic AIS data with historical patterns (e.g., Merak-Bakauheni congestion metrics).  
  - Test dashboard alerts for accuracy (e.g., 92% congestion forecasting [[001_Dashboard_Requirements_Deep_Analysis_.md]].  
- **Extensibility Roadmap**:  
  - Phase 1: Synthetic data for SPCC dashboard PoC.  
  - Phase 2: Integrate real AIS/VDES data streams [[004_SPCC_Insight_Doc.md]].  

---

### **INPUT DATA**
Since I cannot access external files, please provide:  
- A **summary of key data patterns** (e.g., peak traffic hours in Ferizy, crane vibration thresholds).  
- Or paste **critical snippets** (e.g., AIS data requirements, Ferizy API endpoints).  

---

### **OUTPUT FORMAT**
1. **Simulation Engine**:  
   - Code for generating synthetic data (e.g., Python scripts for AIS/MQTT simulations).  
   - Schema definitions for AIS, Ferizy, and IoT sensor data.  
2. **Dashboard Implementation**:  
   - React/Vite project structure with WebSocket integration.  
   - Sample visualizations (e.g., vessel tracking with Leaflet, equipment health with Recharts).  
3. **Validation & Roadmap**:  
   - Validation checklist (e.g., "Congestion alerts trigger within 5s latency").  
   - Migration plan for transitioning to real data sources.  

---

### **WHY THIS WORKS**
1. **Clarity & Specificity**:  
   - Explicitly defines deliverables (simulation engine, dashboard) and aligns with SPCC technical requirements [[6]].  
   - Breaks down data sources (AIS, Ferizy, IoT) with schema examples.  

2. **Technical Precision**:  
   - Mandates MQTT/CoAP protocols, Apache Spark ML pipelines, and sub-5s latency targets [[4]][[6]].  
   - Recommends Kafka for scalable data streaming [[004_SPCC_Insight_Doc.md]].  

3. **Scalability & Compliance**:  
   - Aligns with ISO 11064 (control room design) and IEEE 2413-2019 (IoT frameworks).  
   - Includes validation steps for SPCC’s predictive analytics goals [[001_Dashboard_Requirements_Deep_Analysis_.md]].  

4. **Educational Value**:  
   - Explains why synthetic data must replicate noise patterns (e.g., 15–30% missing AIS values).  

Would you like the full simulation engine code or dashboard implementation examples? 🚢  