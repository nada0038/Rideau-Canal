# Real-time Monitoring System for Rideau Canal Skateway

This project implements a comprehensive real-time monitoring system for the Rideau Canal Skateway in Ottawa. The system tracks ice conditions at three strategic locations using IoT sensors, Azure cloud services, and a web dashboard to assist the National Capital Commission (NCC) in monitoring skater safety.

**Course:** CST8916 - Remote Data and Real-time Applications  
**Semester:** Fall 2025

## Student Information

- **Name:** Akash Nadackanal Vinod
- **Student ID:** 041156265
- **Course:** CST8916 - Remote Data and Real-time Applications
- **Semester:** Fall 2025

## Repository Links

- **Main Monitoring Repository:** [rideau-canal-monitoring](https://github.com/nada0038/rideau-canal-monitoring)

- **Sensor Simulation Repository:** [rideau-canal-sensor-simulation](https://github.com/nada0038/rideau-canal-sensor-simulation)

- **Web Dashboard Repository:** [rideau-canal-dashboard](https://github.com/nada0038/rideau-canal-dashboard)

- **Video Demonstration:** [YouTube Link](     )

## Project Overview

The Rideau Canal Skateway is one of Ottawa's most popular winter attractions, requiring continuous monitoring to ensure public safety. Manual monitoring methods are inefficient and cannot provide the real-time, data-driven insights necessary for timely decision-making regarding skateway operations. This system addresses this challenge by automatically collecting, processing, and visualizing sensor data from three key monitoring locations: Dow's Lake, Fifth Avenue, and the National Arts Centre (NAC).

## System Objectives

1. **Real-time Data Collection:** Continuously monitor ice conditions at three strategic locations
2. **Data Processing:** Aggregate and analyze sensor data in 5-minute windows
3. **Safety Assessment:** Automatically determine safety status based on ice thickness and temperature thresholds
4. **Data Storage:** Store processed data for dashboard queries and historical analysis
5. **Visualization:** Provide an intuitive web dashboard for monitoring staff and public access

## System Architecture

An architecture diagram is available in the [`architecture\architecture.md`](https://github.com/nada0038/Rideau-Canal/blob/main/architecture/architecture.md).


### Azure Services Utilized

- **Azure IoT Hub** - Device management and message ingestion
- **Azure Stream Analytics** - Real-time stream processing with windowing
- **Azure Cosmos DB** - NoSQL database for low-latency dashboard queries
- **Azure Blob Storage** - Long-term data archival
- **Azure App Service** - Web application hosting and deployment

## Implementation Overview

### 1. IoT Sensor Simulation

A Python-based simulator generates realistic sensor data from three locations, transmitting measurements every 10 seconds. Each sensor message includes:
- Ice thickness (measured in centimeters)
- Surface temperature (measured in Celsius)
- Snow accumulation (measured in centimeters)
- External temperature (measured in Celsius)

The simulator implements gradual value changes to simulate realistic environmental conditions. It connects to Azure IoT Hub using device-specific connection strings for authentication.

**Repository:** [rideau-canal-sensor-simulation](https://github.com/nada0038/rideau-canal-sensor-simulation)  
**Technologies:** Python 3.8+, Azure IoT Device SDK
 
### 2. Azure IoT Hub Configuration

The IoT Hub is configured with three registered devices, one for each monitoring location:
- Device IDs: `dows-lake`, `fifth-avenue`, `nac`
- Tier: Free (F1) or Standard (S1)
- Message routing: Configured to forward messages to Stream Analytics input

Each device has a unique connection string used by the sensor simulator for secure authentication and message transmission.

### 3. Stream Analytics Processing

Azure Stream Analytics processes incoming sensor data using 5-minute tumbling windows. The processing includes:

**Aggregations Calculated:**
- Average, minimum, and maximum ice thickness
- Average, minimum, and maximum surface temperature
- Maximum snow accumulation
- Average external temperature
- Reading count per window

**Safety Status Logic:**
- **Safe:** Ice thickness ≥ 30cm AND Surface temperature ≤ -2°C
- **Caution:** Ice thickness ≥ 25cm AND Surface temperature ≤ 0°C
- **Unsafe:** All other conditions

The Stream Analytics query is located in `https://github.com/nada0038/rideau-canal-monitoring/blob/main/stream-analytics/query.sql` and utilizes Common Table Expressions (CTEs) and windowing functions to organize the processing logic.

**Outputs:**
- Cosmos DB: Real-time aggregated data for dashboard queries
- Blob Storage: Historical data archival for long-term analysis

### 4. Azure Cosmos DB Configuration

Cosmos DB stores the aggregated results for efficient dashboard queries:
- **Database:** `RideauCanalDB`
- **Container:** `SensorAggregations`
- **Partition Key:** `/location` (enables fast queries by location)
- **API:** SQL API
- **Document ID Format:** `{location}-{timestamp}`

This configuration optimizes query performance when retrieving the latest data for each location.

### 5. Azure Blob Storage Configuration

All processed data is archived to Blob Storage for long-term analysis:
- **Container:** `historical-data`
- **Path Pattern:** `aggregations/{date}/{time}`
- **Format:** JSON (line-separated)
- **Purpose:** Permanent record of all sensor readings and aggregations

### 6. Web Dashboard

The web dashboard provides a comprehensive view of the monitoring system:
- Real-time data display for all three locations
- Color-coded safety status badges (Safe/Caution/Unsafe)
- Historical trend charts showing data over the last hour
- Overall system status summary
- Automatic refresh every 30 seconds

The dashboard is built using Node.js, Express.js, and Chart.js, querying Cosmos DB for the latest aggregated data and presenting it in an intuitive format.

**Repository:** [rideau-canal-dashboard](https://github.com/nada0038/rideau-canal-dashboard)  
**Technologies:** Node.js, Express, Azure Cosmos DB SDK, Chart.js

### 7. Azure App Service Deployment

The dashboard is deployed on Azure App Service for public accessibility:
- Continuous deployment configured from GitHub
- Environment variables managed securely through Azure configuration
- HTTPS enabled for secure connections
- Auto-scaling configured for performance

## Screenshots

Screenshots demonstrating the system in operation are available in the `screenshots/` folder:

1. IoT Hub device registry showing three registered devices
2. IoT Hub metrics dashboard displaying message throughput
3. Stream Analytics query editor with the processing query
4. Stream Analytics job status showing running state
5. Cosmos DB Data Explorer with sample aggregated documents
6. Blob Storage container
7. Dashboard running locally with live data


### High-Level Setup Steps

1. **Azure Resources Setup**
   - Created Resource Group
   - Created IoT Hub and register three devices
   - Created Stream Analytics job
   - Created Cosmos DB account, database, and container
   - Created Storage Account and container
   - Created App Service for dashboard deployment

2. **Stream Analytics Configuration**
   - Configured IoT Hub as input
   - Configured Cosmos DB as output
   - Configured Blob Storage as output
   - Deploy query from `stream-analytics/query.sql`
   - Started the Stream Analytics job

3. **Sensor Simulator Setup**
   - See detailed instructions in [sensor simulation repository](https://github.com/nada0038/rideau-canal-sensor-simulation)
   - Obtain device connection strings from IoT Hub
   - Configure environment variables
   - Run the simulator

4. **Dashboard Setup**
   - See detailed instructions in [dashboard repository](https://github.com/nada0038/rideau-canal-dashboard)
   - Obtain Cosmos DB credentials
   - Configure environment variables
   - Run locally or deploy to Azure App Service

5. **Verification**
   - Verify IoT Hub is receiving messages
   - Confirm Stream Analytics is processing data
   - Check that data appears in Cosmos DB
   - Validate dashboard displays current data

## Results and Analysis

### System Performance

The system successfully processes sensor data with the following performance characteristics:
- **Throughput:** Handles approximately 18 messages per minute (3 sensors × 6 messages per minute)
- **Latency:** Data appears in dashboard within 30-60 seconds of sensor reading
- **Query Performance:** Cosmos DB queries respond in under 100ms
- **Reliability:** Stream Analytics maintains consistent processing with minimal errors

### Data Analysis

Each 5-minute aggregation window contains approximately 30 readings per location, providing reliable averages for safety calculations. The safety status logic correctly identifies:
- Safe conditions when ice is sufficiently thick and temperature is adequately cold
- Caution conditions when parameters are borderline
- Unsafe conditions when thresholds are not met

### Sample Outputs

Screenshots in the `screenshots/` folder demonstrate the system's operation, showing data flow from sensors through processing to visualization.

## Challenges and Solutions

### Challenge 1: Stream Analytics Query Complexity

**Problem:** Implementing complex aggregations, windowing functions, and safety logic in a single query presented significant complexity.

**Solution:** The query was structured using Common Table Expressions (CTEs) to break down the logic into manageable sections: data cleaning, aggregation calculation, and safety status determination. This approach improved maintainability and debugging.

### Challenge 2: Dashboard Real-time Updates

**Problem:** Ensuring the dashboard displays the most current data without requiring manual page refresh.

**Solution:** Implemented an auto-refresh mechanism using JavaScript's `setInterval` with 30-second intervals. Added comprehensive error handling to ensure the dashboard continues functioning even when API calls fail, gracefully retrying on subsequent refresh cycles.

### Challenge 3: Azure Service Integration

**Problem:** Configuring multiple Azure services to work together seamlessly required careful attention to connection strings, endpoints, and data formats.

**Solution:** Each service integration was tested individually before connecting to the next. Configuration was verified step-by-step, and Azure documentation was consulted for proper setup procedures.

### Challenge 4: Data Format Consistency

**Problem:** Ensuring sensor data format matches Stream Analytics input expectations exactly.

**Solution:** Implemented strict JSON schema validation in the sensor simulator and added error handling in Stream Analytics to catch format mismatches early in the pipeline.

### Challenge 5: Development Environment Issues

**Problem:** Encountered VS Code crashes and configuration issues that interrupted development workflow.

**Solution:** Utilized AI-assisted tools (ChatGPT) to help debug specific errors, learn Azure CLI commands, and understand code formatting requirements. This assistance was limited to troubleshooting specific issues rather than generating core functionality.

## AI Tools Disclosure

### Tools Used

- **ChatGPT** - Used for:
  - Debugging specific errors when development environment crashed
  - Learning Azure CLI command syntax
  - Understanding code formatting and best practices
  - Troubleshooting configuration issues

### Extent of AI Assistance

**AI-Generated Components:**
- Debugging assistance for specific errors
- Command syntax examples
- Code formatting guidance
- Documentation structure templates

**Original Work:**
- All business logic and safety status calculations
- Stream Analytics query design and implementation
- Dashboard UI design and functionality
- Azure service configuration and integration
- System testing and debugging
- Complete understanding of all system components

### Understanding and Verification

All code has been thoroughly reviewed, tested, and understood. I can explain every component of the system, including the Stream Analytics query logic, sensor simulation algorithms, dashboard API endpoints, and Azure service configurations. AI tools were used as a learning aid and troubleshooting assistant, not as a primary development tool.

## References

### Libraries and Frameworks

**Python:**
- `azure-iot-device` - Azure IoT Hub SDK for device communication
- `python-dotenv` - Environment variable management

**Node.js:**
- `express` - Web server framework
- `@azure/cosmos` - Azure Cosmos DB SDK
- `cors` - Cross-origin resource sharing middleware
- `dotenv` - Environment variable management

**Frontend:**
- `Chart.js` - Data visualization library
- `chartjs-adapter-date-fns` - Time-based chart axis support

### Azure Documentation

- [Azure IoT Hub Documentation](https://docs.microsoft.com/azure/iot-hub/)
- [Azure Stream Analytics Documentation](https://docs.microsoft.com/azure/stream-analytics/)
- [Azure Cosmos DB Documentation](https://docs.microsoft.com/azure/cosmos-db/)
- [Azure App Service Documentation](https://docs.microsoft.com/azure/app-service/)

