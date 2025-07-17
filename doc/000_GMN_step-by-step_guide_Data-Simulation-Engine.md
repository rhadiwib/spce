I understand you're looking for a detailed, step-by-step guide to implement the simulation engine and dashboard. While I cannot provide full, runnable code for a complex system like the Smart Port Command Center PoC, I can certainly guide you through the high-fidelity steps, outlining the architecture, key components, and the Python-based approach you can take to build it.

This will involve setting up your environment, developing the data generation logic for each source (AIS, Ferizy, IoT), streaming that data, building a backend API to serve it, and finally, a conceptual overview of the frontend dashboard.

Let's break down the process:

### 1\. Environment Setup

The foundation of your real-time data simulation will be a message broker and a Python development environment.

  * **1.1 Docker Compose for Apache Kafka (and Zookeeper)**
    Apache Kafka is an excellent choice for handling high-volume, high-velocity, continuous data streams, which is essential for real-time simulations.[6, 7] Zookeeper is typically used by Kafka for coordination.

    Create a `docker-compose.yml` file:

    ```yaml
    version: '3.8'
    services:
      zookeeper:
        image: confluentinc/cp-zookeeper:latest
        hostname: zookeeper
        container_name: zookeeper
        ports:
          - "2181:2181"
        environment:
          ZOOKEEPER_CLIENT_PORT: 2181
          ZOOKEEPER_TICK_TIME: 2000

      kafka:
        image: confluentinc/cp-kafka:latest
        hostname: kafka
        container_name: kafka
        ports:
          - "9092:9092"
          - "29092:29092" # For external access
        environment:
          KAFKA_BROKER_ID: 1
          KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
          KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
          KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
          KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
          KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
          KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true' # Allows topics to be created on the fly [8]
        depends_on:
          - zookeeper
    ```

    To start Kafka and Zookeeper:
    `docker-compose up -d`

  * **1.2 Python Environment Setup**
    You'll need Python installed (version 3.7+ is recommended [9]). Then, install the necessary libraries:

    ```bash
    pip install kafka-python Faker Flask Flask-Cors
    ```

      * `kafka-python`: For interacting with Kafka.[10, 11]
      * `Faker`: For generating realistic fake data.[8]
      * `Flask`: For building the web backend API.[12]
      * `Flask-Cors`: To handle Cross-Origin Resource Sharing for your frontend.[12]

### 2\. Data Generation Modules (Python)

This is the core of your simulation engine. You'll create separate Python scripts or classes for each data source to maintain modularity.[13]

  * **2.1 AIS Data Generator (`ais_simulator.py`)**
    This module will simulate vessel movements and static information.

      * **Static Data:** Use `Faker` to generate unique MMSI numbers, vessel names, call signs, and assign realistic ship types.[14, 15, 16] Ensure consistency for a given vessel ID.[14]
      * **Dynamic Data (Trajectory Simulation):**
          * **Initial State:** Define a starting latitude, longitude, speed, and course for each simulated vessel.
          * **Movement Logic:** Implement logic to simulate different navigational statuses:
              * **Underway (Normal Navigation):** Consistent speed and course, with small random variations.[8, 17]
              * **Maneuvering:** Introduce changes in speed and course, simulating turns (realistic `rateOfTurn`).[14, 17]
              * **At Anchor/Moored:** Speed near zero, specific navigational status.[14]
          * **Time-Series Generation:** Use `Faker`'s date/time providers to generate realistic timestamps.[18, 19] Ensure the update frequency varies based on the vessel's activity (e.g., every 10 seconds when underway, every 3 minutes at anchor).[20, 14]
          * **Anomalies:** Intentionally introduce noise, positional errors (up to 116.9m [21]), or data gaps to mimic real-world imperfections.[22, 23] Simulate unexpected turns, sudden stops, or loitering in restricted areas to test anomaly detection.[24, 25]

    **Conceptual `AISVesselSimulator` Class Structure:**

    ```python
    import time
    import json
    from faker import Faker
    from datetime import datetime, timedelta
    import random

    class AISVesselSimulator:
        def __init__(self, mmsi, initial_lat, initial_lon, ship_type, name, call_sign):
            self.fake = Faker()
            self.mmsi = mmsi
            self.latitude = initial_lat
            self.longitude = initial_lon
            self.ship_type = ship_type
            self.name = name
            self.call_sign = call_sign
            self.speed_over_ground = 0.0 # knots
            self.course_over_ground = 0.0 # degrees
            self.true_heading = 0 # degrees
            self.navigational_status = 5 # 5 = moored [14]
            self.rate_of_turn = -128 # No turn info available [14]
            self.last_update_time = datetime.utcnow()
            self.destination = self.fake.city()
            self.eta = (datetime.utcnow() + timedelta(days=random.randint(1, 7))).strftime("%m%d%H%M") # MMDDHHMM UTC [14]
            self.draught = round(random.uniform(3.0, 15.0), 1) # meters [14]

        def _generate_next_position(self):
            # Simple movement logic: move towards a random point, then pick a new one
            # For high-fidelity, this would involve more complex trajectory algorithms [26, 8, 22]
            if self.navigational_status == 5: # Moored
                self.speed_over_ground = 0.0
                self.rate_of_turn = -128
                self.true_heading = self.course_over_ground # Heading aligns with COG when static
                return

            # Simulate movement
            move_distance = self.speed_over_ground * (random.uniform(0.5, 1.5) / 3600) # knots to degrees approx
            self.latitude += move_distance * math.cos(math.radians(self.course_over_ground))
            self.longitude += move_distance * math.sin(math.radians(self.course_over_ground))

            # Introduce slight random variations
            self.speed_over_ground = max(0.0, self.speed_over_ground + random.uniform(-0.1, 0.1))
            self.course_over_ground = (self.course_over_ground + random.uniform(-5, 5)) % 360
            self.true_heading = (self.true_heading + random.uniform(-5, 5)) % 360

            # Simulate navigational status changes (e.g., leaving port, entering port)
            if random.random() < 0.01: # 1% chance to change status
                self.navigational_status = random.choice() # Underway, at anchor, etc. [14]
                if self.navigational_status == 0: # Underway
                    self.speed_over_ground = random.uniform(5.0, 20.0)
                elif self.navigational_status == 1 or self.navigational_status == 5: # At anchor/moored
                    self.speed_over_ground = 0.0
            
            # Simulate rate of turn for maneuvering [14]
            if abs(self.course_over_ground - self.true_heading) > 10 and self.speed_over_ground > 1:
                self.rate_of_turn = random.randint(-126, 126)
            else:
                self.rate_of_turn = -128 # No turn info

            # Introduce positional error [21]
            self.latitude += random.uniform(-0.0001, 0.0001) # ~11m error
            self.longitude += random.uniform(-0.0001, 0.0001)

        def generate_ais_message(self):
            self._generate_next_position()
            current_time = datetime.utcnow()
            self.last_update_time = current_time

            message = {
                "mmsi": self.mmsi,
                "timestamp": current_time.isoformat() + "+00:00", # UTC format [27]
                "latitude": round(self.latitude, 6),
                "longitude": round(self.longitude, 6),
                "courseOverGround": round(self.course_over_ground, 1),
                "speedOverGround": round(self.speed_over_ground, 1),
                "trueHeading": self.true_heading,
                "navigationalStatus": self.navigational_status,
                "imoNumber": self.fake.unique.random_int(min=1000000, max=9999999), # Example IMO [14]
                "name": self.name,
                "callSign": self.call_sign,
                "shipType": self.ship_type,
                "draught": self.draught,
                "destination": self.destination,
                "eta": self.eta,
                "rateOfTurn": self.rate_of_turn
            }
            return message

    # Example Usage (in a separate producer script)
    # vessel1 = AISVesselSimulator(234567890, 51.34, 3.81, 70, "CARGO SHIP A", "CSHA")
    # ais_message = vessel1.generate_ais_message()
    # print(json.dumps(ais_message, indent=2))
    ```

  * **2.2 Ferizy Data Generator (`ferizy_simulator.py`)**
    This module will simulate ferry bookings, passenger manifests, and operational updates, with a focus on ODOL (Over-Dimension and Over-Load) detection.[28]

      * **Booking Information:** Generate unique booking IDs, customer details (using `Faker`), route IDs, departure/arrival times, and assigned vessels.[29, 30]
      * **Ticket Types & Pricing:** Simulate different ticket types (passenger, vehicle, cargo) and dynamic pricing based on demand (e.g., higher prices for peak times).[29]
      * **Vehicle Details (ODOL):** Crucially, include fields for vehicle type, dimensions (length, width, height), and weight. Introduce scenarios where some vehicles exceed predefined ODOL limits to test the dashboard's alert capabilities.[28]
      * **Operational Updates:** Simulate check-in events, cancellations, and real-time vessel occupancy updates.[29, 30]

    **Conceptual `FerizyBookingSimulator` Class Structure:**

    ```python
    import json
    from faker import Faker
    from datetime import datetime, timedelta
    import random

    class FerizyBookingSimulator:
        def __init__(self):
            self.fake = Faker()
            self.routes = {
                "MERAK-BAKAUHENI": {"duration_hours": 2, "max_vehicles": 100, "max_passengers": 500},
                "KETAPANG-GILIMANUK": {"duration_hours": 1.5, "max_vehicles": 80, "max_passengers": 400}
            }
            self.vessels = {
                "VESSEL_F1": {"route": "MERAK-BAKAUHENI", "capacity": {"vehicles": 100, "passengers": 500}},
                "VESSEL_F2": {"route": "MERAK-BAKAUHENI", "capacity": {"vehicles": 100, "passengers": 500}},
                "VESSEL_F3": {"route": "KETAPANG-GILIMANUK", "capacity": {"vehicles": 80, "passengers": 400}}
            }
            self.odol_limits = {"length": 12.0, "width": 2.5, "height": 4.2, "weight": 40000} # Example limits in meters/kg

        def generate_booking(self, booking_id):
            route_id = random.choice(list(self.routes.keys()))
            route_info = self.routes[route_id]
            
            departure_time = self.fake.date_time_between(start_date='now', end_date='+7d', tzinfo=None)
            arrival_time = departure_time + timedelta(hours=route_info["duration_hours"])
            
            vessel_id = random.choice([v_id for v_id, v_info in self.vessels.items() if v_info["route"] == route_id])
            
            ticket_type = random.choice(["passenger", "vehicle", "cargo"])
            num_passengers = 0
            vehicle_details = None
            cargo_details = None
            
            if ticket_type == "passenger":
                num_passengers = random.randint(1, 5)
            elif ticket_type == "vehicle":
                num_passengers = random.randint(1, 3)
                vehicle_length = round(random.uniform(4.0, 15.0), 1)
                vehicle_width = round(random.uniform(1.8, 3.0), 1)
                vehicle_height = round(random.uniform(1.5, 4.5), 1)
                vehicle_weight = random.randint(1500, 60000) # kg

                # Introduce ODOL anomalies [28]
                if random.random() < 0.15: # 15% chance of ODOL
                    odol_type = random.choice(["length", "width", "height", "weight"])
                    if odol_type == "length": vehicle_length = round(random.uniform(self.odol_limits["length"] + 0.5, self.odol_limits["length"] + 5.0), 1)
                    elif odol_type == "width": vehicle_width = round(random.uniform(self.odol_limits["width"] + 0.1, self.odol_limits["width"] + 0.5), 1)
                    elif odol_type == "height": vehicle_height = round(random.uniform(self.odol_limits["height"] + 0.1, self.odol_limits["height"] + 1.0), 1)
                    elif odol_type == "weight": vehicle_weight = random.randint(self.odol_limits["weight"] + 5000, self.odol_limits["weight"] + 20000)

                vehicle_details = {
                    "type": random.choice(["car", "truck", "bus", "motorcycle"]),
                    "dimensions": {"length": vehicle_length, "width": vehicle_width, "height": vehicle_height},
                    "weight_kg": vehicle_weight,
                    "is_odol": (vehicle_length > self.odol_limits["length"] or
                                vehicle_width > self.odol_limits["width"] or
                                vehicle_height > self.odol_limits["height"] or
                                vehicle_weight > self.odol_limits["weight"])
                }
            elif ticket_type == "cargo":
                cargo_details = {
                    "type": random.choice(["container", "bulk", "liquid"]),
                    "weight_kg": random.randint(1000, 50000),
                    "volume_m3": round(random.uniform(5.0, 100.0), 1)
                }

            price = round(random.uniform(50.0, 500.0) * (1 + (random.random() * 0.2 if departure_time.hour in [1, 2, 3, 4] else 0)), 2) # Dynamic pricing [29]

            booking = {
                "booking_id": f"BOOK-{booking_id:05d}",
                "customer_id": self.fake.uuid4(),
                "customer_name": self.fake.name(),
                "customer_email": self.fake.email(),
                "route_id": route_id,
                "vessel_id": vessel_id,
                "departure_datetime": departure_time.isoformat(),
                "arrival_datetime": arrival_time.isoformat(),
                "ticket_type": ticket_type,
                "number_of_passengers": num_passengers,
                "vehicle_details": vehicle_details,
                "cargo_details": cargo_details,
                "price": price,
                "status": "booked" # Initial status
            }
            return booking

        def update_booking_status(self, booking):
            # Simulate check-in, cancellation, etc.
            if booking["status"] == "booked" and random.random() < 0.2:
                booking["status"] = "checked-in"
            elif booking["status"] == "booked" and random.random() < 0.05:
                booking["status"] = "cancelled"
            return booking

    # Example Usage (in a separate producer script)
    # ferizy_sim = FerizyBookingSimulator()
    # booking_data = ferizy_sim.generate_booking(1)
    # print(json.dumps(booking_data, indent=2))
    ```

  * **2.3 IoT Sensor Data Generator (`iot_simulator.py`)**
    This module will simulate various types of IoT sensor data, including environmental and operational sensors.[31, 32]

      * **Sensor Types:** Include temperature, humidity, wind speed/direction, water level, load cells, and GPS for assets.[31, 32]
      * **Time-Series Data:** Generate continuous time-series data that reflects typical patterns, trends, and seasonalities.[33, 34] Libraries like `ydata-sdk` or `deepecho` (though more complex for direct simulation) can inspire the logic.[33, 34, 35]
      * **Realistic Ranges & Frequencies:** Ensure data stays within plausible ranges (e.g., temperature in a refrigerated container) and update frequencies (e.g., meteorological sensors every 3 minutes [26]).
      * **Anomalies & Events:** Crucially, simulate anomalies like sudden temperature spikes, abnormal vibrations in equipment, or power outages.[36, 30, 32, 37] These should trigger "alert" conditions that the dashboard needs to detect.

    **Conceptual `IoTSensorSimulator` Class Structure:**

    ```python
    import json
    from faker import Faker
    from datetime import datetime, timedelta
    import random
    import math

    class IoTSensorSimulator:
        def __init__(self, sensor_id, sensor_type, location, unit, min_val, max_val, anomaly_threshold=None):
            self.fake = Faker()
            self.sensor_id = sensor_id
            self.sensor_type = sensor_type
            self.location = location
            self.unit = unit
            self.min_val = min_val
            self.max_val = max_val
            self.current_value = random.uniform(min_val, max_val)
            self.last_update_time = datetime.utcnow()
            self.anomaly_threshold = anomaly_threshold # e.g., (lower_bound, upper_bound) for anomalies
            self.is_anomalous = False

        def _generate_next_value(self):
            # Simulate natural fluctuation
            change = random.uniform(-0.5, 0.5) * (self.max_val - self.min_val) / 20
            self.current_value = max(self.min_val, min(self.max_val, self.current_value + change))

            # Introduce anomalies [36, 30, 32, 37]
            if self.anomaly_threshold and random.random() < 0.05: # 5% chance of anomaly
                if not self.is_anomalous: # Start anomaly
                    self.is_anomalous = True
                    self.current_value = random.uniform(self.anomaly_threshold, self.anomaly_threshold[5])
                else: # Continue anomaly
                    self.current_value = random.uniform(self.anomaly_threshold, self.anomaly_threshold[5])
                    if random.random() < 0.2: # 20% chance to end anomaly
                        self.is_anomalous = False
            elif self.is_anomalous: # If anomaly was active but didn't continue, revert to normal
                self.is_anomalous = False
                self.current_value = random.uniform(self.min_val, self.max_val)

            # Specific logic for GPS (simulating asset movement)
            if self.sensor_type == "GPS":
                # Simple random walk for GPS, for realism, link to AIS vessel movements
                self.current_value = {
                    "latitude": self.current_value["latitude"] + random.uniform(-0.00005, 0.00005),
                    "longitude": self.current_value["longitude"] + random.uniform(-0.00005, 0.00005),
                    "speed_kph": random.uniform(0.1, 10.0)
                }
                self.current_value["latitude"] = max(-90.0, min(90.0, self.current_value["latitude"]))
                self.current_value["longitude"] = max(-180.0, min(180.0, self.current_value["longitude"]))


        def generate_sensor_reading(self):
            self._generate_next_value()
            current_time = datetime.utcnow()
            self.last_update_time = current_time

            reading = {
                "sensor_id": self.sensor_id,
                "sensor_type": self.sensor_type,
                "location": self.location,
                "timestamp": current_time.isoformat() + "+00:00",
                "value": round(self.current_value, 2) if isinstance(self.current_value, (float, int)) else self.current_value,
                "unit": self.unit,
                "is_anomalous": self.is_anomalous
            }
            return reading

    # Example Usage (in a separate producer script)
    # temp_sensor = IoTSensorSimulator("TEMP-001", "Temperature", "Container 1A", "°C", 20.0, 25.0, anomaly_threshold=(30.0, 35.0))
    # gps_sensor = IoTSensorSimulator("GPS-CRANE-001", "GPS", "Yard Crane 3", "coords", {"latitude": 51.34, "longitude": 3.81}, None)
    # reading = temp_sensor.generate_sensor_reading()
    # print(json.dumps(reading, indent=2))
    ```

### 3\. Data Streaming (Python Kafka Producers)

Each data generator will act as a producer, sending its simulated data to a dedicated Kafka topic.

  * **3.1 Generic Kafka Producer (`kafka_producer.py`)**
    This script will instantiate your simulators and continuously send data to Kafka.

    ```python
    from kafka import KafkaProducer
    import json
    import time
    import random
    import math # For AIS simulator

    # Import your simulator classes
    from ais_simulator import AISVesselSimulator
    from ferizy_simulator import FerizyBookingSimulator
    from iot_simulator import IoTSensorSimulator

    # Kafka configuration
    KAFKA_BROKER = 'localhost:29092' # Use the external port from docker-compose [8, 10, 11]

    producer = KafkaProducer(
        bootstrap_servers=KAFKA_BROKER,
        value_serializer=lambda v: json.dumps(v).encode('utf-8'), # Serialize JSON to bytes [8]
        key_serializer=lambda v: str(v).encode('utf-8') # Keys should also be bytes
    )

    # Initialize Simulators
    # AIS Simulators
    vessels =

    # Ferizy Simulator
    ferizy_sim = FerizyBookingSimulator()
    booking_counter = 0

    # IoT Simulators
    iot_sensors =

    print("Starting data generation and streaming to Kafka...")

    try:
        while True:
            # Generate and send AIS data
            for vessel in vessels:
                ais_message = vessel.generate_ais_message()
                producer.send('ais_data', key=ais_message['mmsi'], value=ais_message)
                # print(f"Sent AIS: {ais_message['mmsi']}")

            # Generate and send Ferizy data
            if random.random() < 0.5: # Simulate new bookings periodically
                booking_counter += 1
                booking_data = ferizy_sim.generate_booking(booking_counter)
                producer.send('ferizy_data', key=booking_data['booking_id'], value=booking_data)
                # print(f"Sent Ferizy Booking: {booking_data['booking_id']}")
            
            # Simulate updates to existing bookings (e.g., check-in)
            # This would require storing and retrieving active bookings, more complex for a simple example
            # For now, new bookings are the primary stream.

            # Generate and send IoT data
            for sensor in iot_sensors:
                sensor_reading = sensor.generate_sensor_reading()
                producer.send('iot_data', key=sensor_reading['sensor_id'], value=sensor_reading)
                # print(f"Sent IoT: {sensor_reading['sensor_id']}")

            producer.flush() # Ensure messages are sent [10]
            time.sleep(1) # Adjust sleep time to control data volume and frequency
    except KeyboardInterrupt:
        print("Stopping data generation.")
    finally:
        producer.close()
    ```

### 4\. Backend API (Python Flask with Server-Sent Events - SSE)

The Flask backend will consume data from Kafka topics and push it to the frontend dashboard using Server-Sent Events (SSE). SSE is ideal for unidirectional, server-to-client real-time updates.[12]

  * **4.1 Flask Application (`app.py`)**

    ```python
    from flask import Flask, Response, stream_with_context
    from flask_cors import CORS
    from kafka import KafkaConsumer
    import json
    import time
    import threading

    app = Flask(__name__)
    CORS(app) # Enable CORS for frontend communication [12]

    KAFKA_BROKER = 'localhost:29092' # Use the external port from docker-compose

    # Kafka Consumers (one for each topic)
    # Ensure these are created only once and managed properly in a real app
    # For simplicity, we'll create them in the generator function, but in production,
    # you'd want a more robust consumer management strategy.

    def get_kafka_consumer(topic):
        return KafkaConsumer(
            topic,
            bootstrap_servers=KAFKA_BROKER,
            auto_offset_reset='latest', # Start consuming from the latest message
            enable_auto_commit=True,
            group_id=f'dashboard-consumer-{topic}',
            value_deserializer=lambda x: json.loads(x.decode('utf-8'))
        )

    @app.route('/stream/ais')
    def stream_ais_data():
        def generate():
            consumer = get_kafka_consumer('ais_data')
            for message in consumer:
                yield f"data: {json.dumps(message.value)}\n\n"
            consumer.close()
        return Response(stream_with_context(generate()), content_type='text/event-stream')

    @app.route('/stream/ferizy')
    def stream_ferizy_data():
        def generate():
            consumer = get_kafka_consumer('ferizy_data')
            for message in consumer:
                yield f"data: {json.dumps(message.value)}\n\n"
            consumer.close()
        return Response(stream_with_context(generate()), content_type='text/event-stream')

    @app.route('/stream/iot')
    def stream_iot_data():
        def generate():
            consumer = get_kafka_consumer('iot_data')
            for message in consumer:
                yield f"data: {json.dumps(message.value)}\n\n"
            consumer.close()
        return Response(stream_with_context(generate()), content_type='text/event-stream')

    if __name__ == '__main__':
        # Run Flask in threaded mode to handle multiple SSE connections [38]
        app.run(debug=True, port=5000, threaded=True)
    ```

### 5\. Frontend Dashboard (Conceptual - React/JavaScript)

The frontend will be a web application that connects to your Flask SSE endpoints to receive real-time data and visualize it.

  * **5.1 React Component Structure (Conceptual)**
    You would typically use a JavaScript framework like React (as mentioned in your previous context) to build the dashboard.

    ```javascript
    // src/App.js (or a dedicated component like Dashboard.js)
    import React, { useEffect, useState } from 'react';

    function App() {
      const = useState();
      const = useState();
      const = useState();

      useEffect(() => {
        // AIS Stream
        const aisEventSource = new EventSource('http://localhost:5000/stream/ais');
        aisEventSource.onmessage = (event) => {
          const newData = JSON.parse(event.data);
          setAisData(prevData =>.slice(-100)); // Keep last 100 for display
        };
        aisEventSource.onerror = (error) => {
          console.error('AIS SSE connection error:', error);
          aisEventSource.close();
        };

        // Ferizy Stream
        const ferizyEventSource = new EventSource('http://localhost:5000/stream/ferizy');
        ferizyEventSource.onmessage = (event) => {
          const newData = JSON.parse(event.data);
          setFerizyData(prevData =>.slice(-50)); // Keep last 50
        };
        ferizyEventSource.onerror = (error) => {
          console.error('Ferizy SSE connection error:', error);
          ferizyEventSource.close();
        };

        // IoT Stream
        const iotEventSource = new EventSource('http://localhost:5000/stream/iot');
        iotEventSource.onmessage = (event) => {
          const newData = JSON.parse(event.data);
          setIotData(prevData =>.slice(-200)); // Keep last 200
        };
        iotEventSource.onerror = (error) => {
          console.error('IoT SSE connection error:', error);
          iotEventSource.close();
        };

        // Cleanup on component unmount
        return () => {
          aisEventSource.close();
          ferizyEventSource.close();
          iotEventSource.close();
        };
      },); // Empty dependency array means this runs once on mount

      return (
        <div className="App">
          <h1>Smart Port Command Center PoC</h1>

          <section>
            <h2>AIS Data (Vessel Movements)</h2>
            {/* Example: Display latest AIS data in a list or on a map */}
            {aisData.map((vessel, index) => (
              <div key={index}>
                {vessel.name} (MMSI: {vessel.mmsi}): Lat {vessel.latitude}, Lon {vessel.longitude}, Speed {vessel.speedOverGround} knots
              </div>
            ))}
          </section>

          <section>
            <h2>Ferizy Data (Bookings & ODOL)</h2>
            {/* Example: Display latest Ferizy bookings, highlight ODOL */}
            {ferizyData.map((booking, index) => (
              <div key={index} style={{ color: booking.vehicle_details?.is_odol? 'red' : 'inherit' }}>
                Booking ID: {booking.booking_id}, Route: {booking.route_id}, Status: {booking.status}
                {booking.vehicle_details?.is_odol && <strong> (ODOL Detected!)</strong>}
              </div>
            ))}
          </section>

          <section>
            <h2>IoT Sensor Data</h2>
            {/* Example: Display latest sensor readings, highlight anomalies */}
            {iotData.map((sensor, index) => (
              <div key={index} style={{ color: sensor.is_anomalous? 'orange' : 'inherit' }}>
                Sensor: {sensor.sensor_type} ({sensor.sensor_id}) at {sensor.location}: {sensor.value} {sensor.unit}
                {sensor.is_anomalous && <strong> (Anomaly!)</strong>}
              </div>
            ))}
          </section>
        </div>
      );
    }

    export default App;
    ```

    To run a React app:
    `npx create-react-app smart-port-dashboard`
    `cd smart-port-dashboard`
    Copy the `App.js` content into `src/App.js` and then run:
    `npm start`

### High-Fidelity Step-by-Step Process Summary:

1.  **Set up Docker Compose:** Save the `docker-compose.yml` and run `docker-compose up -d` to get Kafka and Zookeeper running.
2.  **Install Python Libraries:** Run `pip install kafka-python Faker Flask Flask-Cors`.
3.  **Create Data Generator Files:**
      * `ais_simulator.py`: Implement the `AISVesselSimulator` class with logic for static data, dynamic movement (including speed, course, rate of turn, navigational status), and realistic update frequencies.
      * `ferizy_simulator.py`: Implement the `FerizyBookingSimulator` class to generate bookings, customer info, and crucially, vehicle dimensions/weights to simulate ODOL scenarios.
      * `iot_simulator.py`: Implement the `IoTSensorSimulator` class for various sensor types, generating time-series data with natural fluctuations and injecting anomalies (e.g., temperature spikes, load cell drops).
4.  **Create Kafka Producer Script:**
      * `kafka_producer.py`: Import your simulator classes. Instantiate multiple instances of `AISVesselSimulator` and `IoTSensorSimulator`, and one `FerizyBookingSimulator`.
      * In a continuous loop, call the `generate_message` (or similar) method on each simulator instance.
      * Use `KafkaProducer` to send the generated JSON data to respective Kafka topics (e.g., `ais_data`, `ferizy_data`, `iot_data`). Remember to `flush()` the producer periodically.
      * Run this script: `python kafka_producer.py`.
5.  **Create Flask Backend API:**
      * `app.py`: Set up a Flask application.
      * Define SSE endpoints (`/stream/ais`, `/stream/ferizy`, `/stream/iot`).
      * In each endpoint, create a `KafkaConsumer` for the relevant topic.
      * Use a Python `generator` function with `stream_with_context` to continuously yield messages from Kafka as SSE events.
      * Run this Flask app: `python app.py`.
6.  **Develop React Frontend (Conceptual):**
      * Create a new React project.
      * In your main component (`App.js`), use the `EventSource` API to connect to each of your Flask SSE endpoints.
      * When data is received, update the component's state.
      * Render the received data on the dashboard. For AIS, consider a map visualization. For Ferizy, use tables with conditional formatting for ODOL alerts. For IoT, use charts for time-series data and visual indicators for anomalies.

### Key Considerations for High-Fidelity:

  * **Modularity:** Keep your data generation logic separate from your streaming and API logic. This makes it easier to test, debug, and extend.[13, 39]
  * **Realism and Variability:** Continuously refine your data generation logic. Don't just generate random numbers; think about the statistical distributions, interdependencies between fields, and realistic operational patterns.[40, 2, 25]
  * **Anomalies and Edge Cases:** Explicitly design scenarios for anomalies (e.g., a vessel deviating from its course, an ODOL vehicle attempting to board, a sensor reading out of range).[24, 40, 25] This is crucial for testing the "Command Center" aspect of your dashboard.
  * **Temporal Fidelity:** Ensure timestamps are accurate and update frequencies reflect real-world behavior. This is vital for real-time dashboards.[20, 14, 26]
  * **Scalability:** While this PoC uses a single Kafka instance, remember that Kafka is designed for high throughput and can scale to handle massive data volumes.[7] Your Python producers and consumers should be efficient.
  * **Validation:** As you build, validate your synthetic data. Does it look and behave like real data? Does it trigger the dashboard's features as expected?.[40, 41, 2, 25, 42]

This detailed guide provides a robust framework for building your Smart Port Command Center PoC. Remember, this is a significant undertaking, and each step will require careful implementation and iterative refinement. Good luck\!