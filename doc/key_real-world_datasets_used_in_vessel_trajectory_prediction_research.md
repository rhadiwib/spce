Here are key real-world datasets used in vessel trajectory prediction research, categorized by source and application context:

### 1. **Port-Specific Datasets**
#### **Busan Port Inner Harbor (South Korea)** 
- **Source**: Vessel Traffic Service (VTS) records
- **Volume**: 280,000+ AIS records (2023–2024)
- **Features**:
  - Dynamic: Latitude, longitude, SOG, COG, heading
  - Static: Vessel type, MMSI, draft
  - **Berthing side** (port/starboard) for 4A, 4B, and 5A piers
- **Preprocessing**: Linear interpolation for irregular intervals, DBSCAN clustering for trajectory segmentation
- **Use Case**: Predicting final 2 km approach trajectories to piers. Bi-LSTM models reduced RMSE to **107m** for turning maneuvers .

#### **Caofeidian Port (China)**
- **Scope**: Complex port area with mixed vessel types
- **Size**: 45,000 trajectories
- **Metrics**: Tested 12 prediction models (KF, SVR, LSTM, Transformer). Transformers achieved **0.0006 MSE** for <2km predictions .

---

### 2. **Regional Waterways**
#### **Singapore Strait** 
- **Coverage**: 105-km strait (Jan–Dec 2019)
- **Volume**: Avg. 1 million messages/day (800 vessels/min)
- **Key Attributes**: Message types 1, 3, 5, 18, 24; payload decoding required
- **Challenge**: High traffic density causing signal collisions
- **Application**: Anomaly detection for piracy/security threats .

#### **Pearl River Estuary (China)**
- **Data**: April 2023 AIS (1.2M messages)
- **Area**: 21.5°N–23°N, 113.25°E–114.75°E
- **Enrichment**: Integrated weather/tidal data
- **Model**: SeqLSTM-U-Net achieved **MAE 0.0334** for 1-hour predictions
- **Use Case**: Emission calculation via trajectory-based route optimization .

---

### 3. **Global-Scale Datasets**
#### **Worldwide Ports (2011–2017)** 
- **Records**: 141 million AIS messages
- **Trajectories**: 5.9M historical routes between 10,618 ports
- **Preprocessing**: DBSCAN clustering, exclusion of trajectories with >1-day gaps
- **Application**: Random Forest destination prediction (accuracy: **92.8%** for major ports) .

---

### 4. **Model-Specific Benchmark Datasets**
#### **Transformer Training (TPTrans)** 
- **Sources**: Chengshan Jiao Promontory (normal channel), Zhoushan Archipelago (complex traffic)
- **Size**: 650k trajectories
- **Features**: H3 geospatial indexing, Fourier positional encoding
- **Performance**: Reduced MSE by **6×10⁻⁴** vs. LSTM in turn-intensive zones .

#### **Maritime Monitoring Repository**
- **Coverage**: 640M AIS messages (19,016 vessels)
- **Vessel Types**: 
  | **Type**       | **% Short Routes** | **% Long Routes** |
  |----------------|--------------------|-------------------|
  | Passenger      | 85%                | 15%               |
  | Tanker/Cargo   | 22%                | 78%               |
- **Access**: Public GitHub repository ([link](https://github.com/eyesofworld/Maritime-Monitoring)) .

---

### 5. **Specialized Use Cases**
#### **Emission Monitoring (Pearl River)** 
- **Integration**: AIS + engine load + weather data
- **Output**: CO₂ emission estimates linked to predicted routes
- **Impact**: 18% fuel savings via optimized path planning

#### **Collision Avoidance (Bosphorus Strait)**
- **Data**: 22,000 encounter scenarios
- **Model**: Attention-Bi-LSTM with 87.98% recall in high-risk situations
- **Input**: Relative velocity, course difference, azimuth .

---

### Key Insights from Data:
1. **Berthing side** improves prediction accuracy by 15–30% in ports .
2. **Short-voyage vessels** (e.g., ferries) show higher predictability (RMSE <100m) than tankers .
3. **Hybrid datasets** (AIS + weather) reduce long-term prediction errors by 41% during storms .

Public datasets are available via [AIS Hub](http://www.aishub.net/coverage), [FigShare](https://figshare.com/s/67988589931f8c1a6204), and GitHub repositories cited above.