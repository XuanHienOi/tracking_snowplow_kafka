# tracking_snowplow_kafka
tracking a simple web with snowplow and push to kafka


# Tracking Snowplow Events with Kafka (Local Demo Setup)

This project provides a local setup using Docker Compose to demonstrate tracking Snowplow events, collecting them via Snowplow Micro, and forwarding them to a Kafka topic. It includes a simple test website to generate events.

## Prerequisites

*   **Docker:** You need Docker installed and running. [Install Docker](https://docs.docker.com/get-docker/)
*   **Docker Compose:** Typically included with Docker Desktop, but ensure it's available. [Install Docker Compose](https://docs.docker.com/compose/install/)
*   **Node.js & npm:** Required to run the simple `http-server` for the test website. [Install Node.js and npm](https://nodejs.org/)
*   **http-server:** A simple command-line HTTP server. Install it globally:
    ```bash
    npm install --global http-server
    ```
*   **Git:** To clone the repository (if applicable).

## Setup

1.  **Clone the Repository (if you haven't already):**
    ```bash
    git clone <your-repository-url>
    cd tracking_snowplow_kafka # Or your project directory name
    ```
2.  **Review Configuration:** Check the `docker-compose.yml` file to understand the services being launched (Snowplow Micro, Kafka, Zookeeper, etc.).

## Running the Application

1.  **Start Services:** Launch Snowplow Micro, Kafka, and other defined services in detached mode.
    ```bash
    docker compose up -d
    ```
    Wait a minute or two for the services to initialize fully, especially Kafka.

2.  **Create Kafka Topic:** Create the necessary Kafka topic (`snowplow-events-raw`) where Snowplow Micro will send the raw events. This command sets a retention period of 1 day (86400000 milliseconds).
    ```bash
    docker compose exec kafka kafka-topics \
      --create \
      --if-not-exists \
      --bootstrap-server kafka:29092 \
      --replication-factor 1 \
      --partitions 1 \
      --topic snowplow-events-raw \
      --config retention.ms=86400000
    ```
    *You should see `Created topic snowplow-events-raw.` or a message indicating it already exists.*

3.  **Start Test Web Server:** In a **new terminal window or tab**, navigate to your project directory (`tracking_snowplow_kafka`) and start the local web server to host the test site.
    ```bash
    # Ensure you are in the project's root directory
    http-server . -p 8080 --cors -c-1
    ```
    *   `-p 8080`: Runs the server on port 8080.
    *   `--cors`: Enables Cross-Origin Resource Sharing headers (important for tracking scripts).
    *   `-c-1`: Disables caching.

## Testing and Monitoring

1.  **Generate Events:**
    *   Open your web browser and navigate to the test site:
        [http://localhost:8080/test_site/](http://localhost:8080/test_site/)
        *(Adjust `/test_site/` if your HTML file is in a different sub-directory or the root)*
    *   Interact with the page (e.g., click buttons, scroll) if it's set up to trigger Snowplow events. Page views should be tracked automatically on load if the tracker is configured correctly in the HTML.

2.  **Monitor Events in Snowplow Micro:**
    *   Open the Snowplow Micro UI in your browser:
        [http://localhost:9090/micro/ui](http://localhost:9090/micro/ui)
    *   You should see events appearing here shortly after you interact with the test site. Check the "Good", "Bad", and "All" event counts.

3.  **Monitor Events in Kafka:**
    *   In **another new terminal window or tab**, run the Kafka console consumer to see the raw events being written to the `snowplow-events-raw` topic:
    ```bash
    docker compose exec kafka kafka-console-consumer \
      --bootstrap-server kafka:29092 \
      --topic snowplow-events-raw \
      --from-beginning
    ```
    *   You should see event data (likely in Snowplow's Thrift or JSON format, depending on Micro's configuration) appearing here as they are processed by Micro and sent to Kafka.
    *   Press `Ctrl+C` to stop the consumer when finished.

## Stopping the Application

1.  **Stop the `http-server`:** Go to the terminal where `http-server` is running and press `Ctrl+C`.
2.  **Stop Docker Services:** Shut down and remove the containers defined in `docker-compose.yml`.
    ```bash
    docker compose down
    ```
    *(Optional: Add `-v` if you also want to remove the Docker volumes associated with the services, like Kafka data)*
