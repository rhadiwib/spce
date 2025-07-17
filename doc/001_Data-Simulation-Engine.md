I need to develop a Proof of Concept (PoC) for the Smart Port Command Center dashboard despite lacking access to critical real-time data sources (AIS, Ferizy, IoT). To enable end-to-end development and testing, I require a comprehensive simulation engine that:

**1. Data Source Replication Requirements:**
   - For each missing data source (AIS/Ferizy/IoT), create *protocol-accurate simulations* that:
     - Mirror actual data schemas (field structure, data types, units)
     - Replicate transmission protocols (NMEA 0183 for AIS, MQTT/CoAP for IoT, REST/GraphQL for Ferizy)
     - Emulate real-world behavior patterns:
        * AIS: Vessel movement physics, navigation status changes
        * Ferizy: Booking surges during peak seasons, no-show patterns
        * IoT: Sensor drift, failure modes, environmental noise

**2. Simulation Engine Capabilities:**
   - Generate synthetic datasets covering:
     - Normal operations (70% of data)
     - Edge cases (20%: storms, congestion, equipment failures)
     - Anomalies (10%: data gaps, protocol violations)
   - Provide real-time data streams at production-scale volumes:
     ```yaml
     AIS: 10,000 msg/min @ 1Hz update rate
     IoT: 50,000 sensor readings/min 
     Ferizy: 500 bookings/min during peak
     ```
   - Include parameter controls for:
     - Data quality degradation (15-30% noise injection)
     - Latency variability (0.1-5s delays)
     - Failure scenario triggers

**3. Fidelity Requirements:**
   - Mathematical models for:
     - Vessel kinematics (position, speed, heading)
     - Passenger demand curves (holiday surges, time-of-day patterns)
     - Equipment degradation profiles
   - Protocol-compliant interfaces:
     ```mermaid
     flowchart LR
         AIS_Sim-->|NMEA 0183|Dashboard
         IoT_Sim-->|MQTT/CoAP|Dashboard
         Ferizy_Sim-->|REST API|Dashboard
     ```

**4. Integration & Testing Features:**
   - Seamless switch between simulated/live data via configuration flag
   - Data validation suite to compare simulation vs production behavior
   - Load testing module for scalability verification (up to 3x peak traffic)

**5. Critical Oversight Considerations:**
   - What metadata requirements (timestamps, source IDs, checksums) are essential?
   - How should we simulate cybersecurity aspects (encrypted payloads, auth tokens)?
   - What regulatory compliance constraints (GDPR, maritime data standards) need simulation?
   - Which performance metrics (latency distributions, error rates) must be replicated?

**Deliverable Goal:** A containerized simulation microservice that enables full dashboard development and presents indistinguishable behavior from real systems during PoC demos.