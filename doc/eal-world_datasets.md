Here are key real-world datasets available in JSON format across multiple domains, with a focus on maritime (AIS) and other fields:

---

### 🌊 **Maritime & AIS Datasets**
1. **DLR Marine Radar & AIS Datasets**  
   - **Content**: Radar point clouds (polar/Cartesian coordinates), AIS ground truth, vessel tracking data.  
   - **Format**: JSON + radar images + Python demo code.  
   - **Access**: Request via DLR contact form (export-controlled; EU/NATO priority) .  
   - **Use Case**: Extended target tracking, collision avoidance algorithms.  

2. **Real-Time AIS APIs**  
   - **VT Explorer**: Live vessel positions, port calls, voyage data in JSON/XML via API .  
   - **AISHub**: Community-shared AIS feeds; JSON/XML/CSV after contributing data .  
   - **VesselFinder**: Raw NMEA or JSON API for vessel tracking (subscription-based) .  

3. **NOAA AIS Vessel Tracks**  
   - **Content**: Historical U.S. coastal vessel trajectories (2020–2024).  
   - **Format**: CSV via Data.gov, but JSON accessible via API integration.  
   - **Access**: Free download .  

---

### 🤖 **Data Science & AI**
1. **Hugging Face Datasets**  
   - **Content**: 40,000+ datasets for NLP, computer vision (e.g., Pile language model corpus).  
   - **Format**: JSON/Parquet via Python library.  
   - **Example**: `load_dataset("pile")` .  

2. **Google AIS Framework**  
   - **Content**: Annotated model outputs for verifiable language generation (CNN/DM, WoW datasets).  
   - **Format**: CSV with JSON-compatible schemas .  

3. **Yelp Open Dataset**  
   - **Content**: 8M+ business reviews, user check-ins.  
   - **Format**: JSON files for academic use .  

---

### 🏛️ **Government & Public Data**
1. **Data.gov Maritime**  
   - 95+ AIS datasets (e.g., vessel densities, transit counts) with JSON API endpoints .  

2. **UK Police Data**  
   - Crime records in England; JSON via REST API .  

3. **NASA Earth Data**  
   - Satellite observations; JSON metadata for dataset discovery .  

---

### 💰 **Finance & Economics**
1. **WorldBank API**  
   - GDP, poverty, climate indicators in JSON .  
2. **CoinMarketCap**  
   - Cryptocurrency prices/market caps via JSON API .  

---

### ⚙️ **Tools & Mock Data**
- **Mockaroo**: Generate custom JSON test data (e.g., simulate vessel trajectories) .  
- **httpbin**: Dynamic HTTP response testing with JSON output .  

---

### 🔑 **Key Insights**
- **Maritime Focus**: DLR datasets are gold standards for radar-AIS fusion but require compliance checks .  
- **Live APIs**: VT Explorer and AISHub offer low-latency JSON for real-time vessel tracking .  
- **Ethical Use**: Anonymize MMSI codes in AIS data to comply with GDPR .  

> 💡 **Tip**: For maritime projects, combine NOAA historical data with real-time APIs to train trajectory prediction models. Use Hugging Face for AI integration examples.