Project topic: Python – gRPC – OTel

Authors: Michał Bert, Jakub Kędra, Aleksandra Sobiesiak, Adrian Stahl

## Introduction

The purpose of this project is to present a modern, high-performance remote procedure call (RPC) framework—gRPC—and its use within the Python ecosystem. The report also covers observability tools for distributed systems, focusing on OpenTelemetry, which enables the collection of metrics, logs, and traces from applications. As a case study, a simple client–server system based on gRPC will be demonstrated, enriched with monitoring mechanisms using OpenTelemetry.

## Case Description & Technology Stack

### gRPC

gRPC is an open-source RPC framework developed by Google that builds on HTTP/2 and uses Protocol Buffers (Protobuf) for data serialization. This provides high performance, low network overhead, and support for advanced features such as:

* bidirectional streaming,
* authentication and security,
* load balancing,
* automatic code generation for multiple programming languages, including Python.

Python makes it easy to create both gRPC servers and clients. With the `grpcio-tools` package, you can generate client and server code from `.proto` files, ensuring consistent and efficient communication between different services and languages.

## Application Description

The application simulates the collection of temperature sensor readings and their transmission to a central server. Both the server and client components are written in Python. For added security, the server uses an SSL certificate.

### Types of RPC Calls / Streaming in gRPC

#### Unary (request → response)

* The client sends a single message; the server returns a single response.

#### Server-streaming (request → stream(response))

* The client sends one request; the server returns a stream of multiple messages.

#### Client-streaming (stream(request) → response)

* The client sends a stream of messages; the server returns a single summary response.

#### Bidirectional-streaming (stream(request) ⇄ stream(response))

* Both sides exchange messages independently and concurrently.

### The `.proto` File

The `.proto` file describes the gRPC service interface (using proto3) and the message formats exchanged between client and server. Below is an overview of its elements:

```proto3
syntax = "proto3";

service SensorService {
  rpc SendSingleReading (SensorReading) returns (Ack);
  rpc StreamSensorReadings (stream SensorReading) returns (Ack);
  rpc GetSensorReadings (SensorRequest) returns (stream SensorReading);
  rpc SensorChat (stream SensorReading) returns (stream ServerMessage);
}
```

* **SendSingleReading**: unary RPC
* **StreamSensorReadings**: client-streaming RPC
* **GetSensorReadings**: server-streaming RPC
* **SensorChat**: bidirectional-streaming RPC

#### Message Definitions

```proto3
message SensorReading {
  string sensor_id = 1;
  double temperature = 2;
  int64 timestamp = 3;
}

message Ack {
  string message = 1;
}

message SensorRequest {
  string sensor_id = 1;
}

message ServerMessage {
  string message = 1;
  int64 server_time = 2;
}
```

* **SensorReading**: represents a sensor measurement.

  * `sensor_id` (1): unique sensor identifier
  * `temperature` (2): temperature value (float)
  * `timestamp` (3): Unix epoch timestamp (int64)
* **Ack**: acknowledgment message with a text field.
* **SensorRequest**: request for readings from a specific sensor.
* **ServerMessage**: message sent by the server in streaming mode, includes a status or alert and server time.

### Server

The server’s responsibilities include collecting sensor data and performing analysis (e.g., computing averages). It implements the contract defined in the `.proto` file as follows:

* **SendSingleReading** – stores the received reading in its in-memory cache.
* **StreamSensorReadings** – same as above but for multiple readings sent as a stream.
* **GetSensorReadings** – returns stored readings to the client as a stream.
* **SensorChat** – receives readings from the client while simultaneously sending back acknowledgments.

### Client

The client simulates sensor behavior, sending measurements to the server. For simulation purposes, four client instances will run, each using a different RPC type described above. Each client periodically sends readings at random intervals or requests stored readings from the server.

### OpenTelemetry (OTel)

OpenTelemetry is a vendor-neutral collection of tools, APIs, and SDKs for instrumenting, collecting, processing, and exporting telemetry data. Its core data types are:

* **Traces** – analyze the flow of requests through a distributed system.
* **Metrics** – numeric data about application performance (e.g., request counts, response times).
* **Logs** – detailed event records from application execution.

OpenTelemetry for Python provides native integration with various frameworks, including gRPC, enabling easy tracing of data flows and real-time monitoring. In this case study, we implement a temperature-sensor data collection system using a client–server architecture with gRPC, exporting telemetry to Jaeger (for traces) and Prometheus/Grafana (for metrics).

## Solution Architecture

![architecture](https://github.com/user-attachments/assets/ecf67907-1df0-4271-b907-5925255b1ae0)

The diagram above shows the main application components and how they interact within a Docker network (`grpc-network`):

* **Black dashed line** – Docker network
* **Colored dashed lines** – Docker containers

## Environment Setup

### Prerequisites

* Python 3.10+
* Docker & Docker Compose (version ≥ 1.29)
* Make
* `grpcurl` client (for manual testing, optional)

The required Python packages are listed in `requirements.txt`:

```
grpcio
grpcio-tools
grpcio-observability
grpcio-health-checking
opentelemetry-sdk
opentelemetry-instrumentation-grpc
opentelemetry-exporter-jaeger
opentelemetry-exporter-otlp
protobuf==3.20.3
```

## Installation & Running the Environment

1. Clone the project locally:

   ```bash
   git clone git@github.com:xramzesx/suu-project.git
   ```

2. Change into the project directory:

   ```bash
   cd suu-projekt/sensor_app
   ```

3. Generate certificates:

   ```bash
   make generate_certs
   ```

4. Start the containers:

   ```bash
   docker compose up --build
   ```

This command will build and start the following containers:

* `grpc-server`
* `grpc-client-1`
* `grpc-client-2`
* `grpc-client-3`
* `otel-collector`
* `prometheus`
* `grafana`
* `jaeger-1`

## Use of AI in the Project

Several queries were made using ChatGPT o4-mini to generate the system architecture diagram from the Docker Compose file, but the results were not sufficiently accurate to use in the project.

## Summary

### Traces in the Jaeger UI

The Jaeger UI is available at [http://localhost:16686](http://localhost:16686) <img width="1423" alt="image" src="https://github.com/user-attachments/assets/93c3655e-d2b8-4aee-82bd-5831eb087818" />

### Metrics in Prometheus

Prometheus is available at [http://localhost:9090](http://localhost:9090) <img width="1423" alt="image" src="https://github.com/user-attachments/assets/5bdf2ce2-efde-40df-92ff-322bd99c6479" /> <img width="1422" alt="image" src="https://github.com/user-attachments/assets/28fffea4-df6f-4409-a958-d0347081e10b" />

### Grafana

Grafana is available at [http://localhost:3000](http://localhost:3000)
Log in with username `admin` and password `admin`.

You can configure Prometheus as a data source for your dashboards: <img width="1427" alt="image" src="https://github.com/user-attachments/assets/fb407b3f-2f4c-4ea0-8c3e-210f1294dfa6" />

Example dashboard query using the `grpc_server_call_duration_seconds_count` metric: <img width="1425" alt="image" src="https://github.com/user-attachments/assets/bff422ae-3af0-4f13-a315-26390b086a37" />

## References

* [https://opentelemetry.io/docs/](https://opentelemetry.io/docs/) – OpenTelemetry Documentation
* [https://grpc.io/docs/languages/python/](https://grpc.io/docs/languages/python/) – gRPC Python Documentation
* [https://prometheus.io/docs/visualization/grafana/](https://prometheus.io/docs/visualization/grafana/) – Prometheus & Grafana Integration
* [https://opentelemetry.io/docs/collector/](https://opentelemetry.io/docs/collector/) – OpenTelemetry Collector Documentation
