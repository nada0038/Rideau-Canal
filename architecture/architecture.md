```mermaid
graph TD
    A[Python Sensor Simulator<br/>3 locations<br/>Every 10 seconds] -->|sends data| B[Azure IoT Hub<br/>3 registered devices]
    B -->|routes messages| C[Azure Stream Analytics<br/>5-min tumbling windows<br/>Aggregates & calculates safety]
    C -->|outputs aggregated data| D[Azure Cosmos DB<br/>RideauCanalDB<br/>SensorAggregations]
    C -->|archives data| E[Azure Blob Storage<br/>historical-data container]
    D -->|queries latest data| F[Web Dashboard<br/>Node.js/Express<br/>Auto-refresh every 30s]
    F -->|deployed on| G[Azure App Service<br/>Public access]
```

## Data Flow

1. **Sensors** : Send data every 10 seconds to IoT Hub
2. **IoT Hub** : Routes messages to Stream Analytics
3. **Stream Analytics** : Processes in 5-minute windows, calculates aggregations and safety status
4. **Cosmos DB** : Stores processed data for dashboard queries
5. **Blob Storage** : Archives all data for historical analysis
6. **Dashboard** : Queries Cosmos DB and displays real-time data
7. **App Service** : Hosts the dashboard for public access

