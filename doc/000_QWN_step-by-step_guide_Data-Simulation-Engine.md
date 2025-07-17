A **high-fidelity, step-by-step guide** to build your **Smart Port Command Center (SPCC) dashboard PoC** with a synthetic data simulation engine, aligned with technical requirements from your documents. Each step includes actionable instructions, technical details, and references to ensure compliance with ISO 11064 (control room design), IEEE 2413-2019 (IoT interoperability), and sub-5s latency targets.

---

### **Phase 1: Build the Synthetic Data Simulation Engine**  
#### **Step 1: Define Data Schemas & Protocols**  
1. **AIS Data Schema**:  
   ```json  
   {  
     "mmsi": "string",  
     "latitude": "number",  
     "longitude": "number",  
     "speed": "number",  
     "timestamp": "datetime"  
   }  
   ```  
   - **Behavior**: Simulate vessel movement patterns (docking, queuing) with 15–30% missing values .  

2. **IoT Sensor Schema**:  
   ```json  
   {  
     "sensor_id": "string",  
     "value": "number",  
     "unit": "string",  
     "timestamp": "datetime"  
   }  
   ```  
   - **Protocols**: Use **MQTT/CoAP** for constrained devices (e.g., crane vibration sensors) .  

3. **Ferizy Booking Schema**:  
   ```json  
   {  
     "vehicle_count": "number",  
     "pedestrian_count": "number",  
     "dynamic_price": "number",  
     "timestamp": "datetime"  
   }  
   ```  
   - **Behavior**: Replicate peak traffic patterns (e.g., Idul Fitri with 3x baseline volume [[002_GMN_GoalsDigitalTransformation.md]].  

#### **Step 2: Implement the Simulation Engine**
1. **AIS Simulation (Python)**:  
   - Use `aispy` to generate AIS messages:  
     ```python  
     import aispy  
     from datetime import datetime  
     def simulate_ais_data():  
         while True:  
             # Generate vessel positions with noise  
             data = {  
                 "mmsi": "538004403",  
                 "latitude": random.uniform(-90, 90),  
                 "longitude": random.uniform(-180, 180),  
                 "speed": random.uniform(0, 30),  
                 "timestamp": str(datetime.now())  
             }  
             # Simulate missing values (15–30%)  
             if random.random() < 0.2:  # 20% missing speed data  
                 del data["speed"]  
             send_to_dashboard(data)  
     ```  

2. **IoT Sensor Simulation (Python + MQTT)**:  
   - Use `paho-mqtt` to simulate sensor data:  
     ```python  
     import paho.mqtt.client as mqtt  
     def simulate_sensor_data():  
         client = mqtt.Client("sensor_simulator")  
         client.connect("localhost")  
         while True:  
             data = {  
                 "sensor_id": "crane_01",  
                 "value": random.uniform(0, 100),  # Vibration in g-force  
                 "unit": "vibration_g",  
                 "timestamp": str(datetime.now())  
             }  
             client.publish("iot/sensor", json.dumps(data))  
     ```  

3. **Ferizy Booking Simulation (REST API)**:  
   - Use Flask to mock booking flows:  
     ```python  
     from flask import Flask, jsonify  
     app = Flask(__name__)  
     @app.route("/api/bookings", methods=["GET"])  
     def mock_bookings():  
         return jsonify({  
             "vehicle_count": random.randint(500, 1000),  
             "pedestrian_count": random.randint(200, 500),  
             "dynamic_price": random.uniform(50, 200),  
             "timestamp": str(datetime.now())  
         })  
     ```  

#### **Step 3: Validate Synthetic Data**
1. **Latency Testing**:  
   - Use **Apache Kafka** to test sub-5s latency:  
     ```bash  
     # Install Kafka  
     brew install kafka  
     # Produce AIS data  
     bin/kafka-console-producer.sh --broker-list localhost:9092 --topic ais_data < ais_simulated.json  
     ```  
   - Consume data in the dashboard with `kafka-node` .  

2. **Accuracy Validation**:  
   - Compare synthetic AIS data with historical congestion metrics (e.g., Merak-Bakauheni peak hours .  

---

### **Phase 2: Build the SPCC Dashboard**  
#### **Step 1: Set Up the React/Vite Project**  
1. **Create React App with Vite**:  
   ```bash  
   npm create vite@latest spcc-dashboard --template react-ts  
   cd spcc-dashboard && npm install  
   ```  

2. **Install Dependencies**:  
   ```bash  
   npm install recharts leaflet react-leaflet axios socket.io-client  
   ```  

#### **Step 2: Implement Core Dashboard Components**
1. **WebSocket Integration**:  
   - Use `useWebSocket` for real-time alerts:  
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

2. **Vessel Tracking with Leaflet**:  
   - Create a map component for AIS data:  
     ```tsx  
     import { MapContainer, TileLayer, Marker, Popup } from "react-leaflet";  
     const VesselMap = ({ aisData }: { aisData: any }) => {  
       return (  
         <MapContainer center={[51.505, -0.09]} zoom={13} scrollWheelZoom={false}>  
           <TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />  
           {aisData.map((vessel: any) => (  
             <Marker position={[vessel.latitude, vessel.longitude]}>  
               <Popup>Speed: {vessel.speed} knots</Popup>  
             </Marker>  
           ))}  
         </MapContainer>  
       );  
     };  
     ```  

3. **Predictive Analytics with Recharts**:  
   - Visualize congestion forecasts:  
     ```tsx  
     import { LineChart, Line, XAxis, YAxis } from "recharts";  
     const CongestionChart = ({ data }: { data: any }) => {  
       return (  
         <LineChart width={600} height={300} data={data}>  
           <Line type="monotone" dataKey="vehicles" stroke="#8884d8" />  
           <XAxis dataKey="hour" />  
           <YAxis />  
         </LineChart>  
       );  
     };  
     ```  

#### **Step 3: Connect Dashboard to Simulation Engine**
1. **Fetch Ferizy Data**:  
   - Use `axios` to consume the Flask API:  
     ```tsx  
     const fetchBookings = async () => {  
       const response = await axios.get("http://localhost:5000/api/bookings");  
       setBookings(response.data);  
     };  
     ```  

2. **Integrate MQTT IoT Data**:  
   - Use `paho-mqtt` for real-time sensor updates:  
     ```tsx  
     const client = new Paho.MQTT.Client("localhost", 8080, "dashboard_client");  
     client.connect({ onSuccess: () => client.subscribe("iot/sensor") });  
     client.onMessageArrived = (msg) => {  
       updateSensorDashboard(JSON.parse(msg.payloadString));  
     };  
     ```  

---

### **Phase 3: Validation & Extensibility**  
#### **Step 1: Validate Against Technical Requirements**  
1. **Latency**:  
   - Test sub-5s latency with Kafka metrics .  
2. **Accuracy**:  
   - Ensure 92% congestion forecasting alignment with synthetic data .  
3. **Security**:  
   - Enable **AES-256 encryption** in synthetic data streams .  

#### **Step 2: Plan for Real Data Integration**
1. **AIS/VDES Transition**:  
   - Replace simulation engine with real AIS/VDES APIs .  
2. **Edge Computing**:  
   - Deploy **NVIDIA Jetson AGX Orin** for sub-5s latency in Phase 2 .  

---

### **Phase 4: Testing & Deployment**  
#### **Step 1: High-Fidelity Prototype Testing**  
- Use **prototype testing** to gather feedback on dashboard usability .  
- Validate against ISO 11064 (control room ergonomics) and IEEE 2413-2019 (IoT interoperability).  

#### **Step 2: Dockerize the System**
1. **Dockerfile for Simulation Engine**:  
   ```dockerfile  
   FROM python:3.10  
   COPY . /app  
   WORKDIR /app  
   RUN pip install -r requirements.txt  
   CMD ["python", "simulate.py"]  
   ```  
2. **Docker Compose**:  
   ```yaml  
   services:  
     dashboard:  
       build: ./spcc-dashboard  
       ports: ["3000:3000"]  
     simulation-engine:  
       build: ./simulation-engine  
       ports: ["5000:5000"]  
     kafka:  
       image: bitnami/kafka:latest  
       ports: ["9092:9092"]  
   ```  

---

### **Critical Gaps & Recommendations**
1. **Noise Patterns**:  
   - AIS data must include missing values (15–30%) to test resilience .  
2. **Edge Cases**:  
   - Simulate sensor failures (e.g., vibration sensor offline) and spoofed AIS signals.  
3. **Security**:  
   - Implement **Zero Trust Architecture** with token-based authentication .  

---

### **Why This Works**
1. **Clarity & Specificity**:  
   - Explicitly defines deliverables (simulation engine, dashboard) and aligns with SPCC technical requirements .  
2. **Technical Precision**:  
   - Mandates MQTT/CoAP protocols, Apache Spark ML pipelines, and sub-5s latency targets .  
3. **Scalability & Compliance**:  
   - Aligns with ISO 11064 (control room design) and IEEE 2413-2019 (IoT frameworks).  

---

### **Example Tools & Libraries**
- **AIS Simulation**: `aispy` (Python AIS message generator) .  
- **IoT Simulation**: `paho-mqtt` (MQTT client for Python).  
- **Dashboard**: **Recharts/Leaflet** for vessel tracking and equipment health monitoring .  

---

### **Final Recommendations**
1. **Start Small**:  
   - Focus on AIS and IoT data first, then add Ferizy integration.  
2. **Prioritize Latency**:  
   - Use Kafka to validate sub-5s latency for critical updates .  
3. **Validate Continuously**:  
   - Compare synthetic data against SPCC’s 92% congestion forecasting accuracy target .  

Would you like the full simulation engine code or dashboard implementation examples? 🚢