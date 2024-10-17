# Assignment-Docker-and-Prometheus-Setup

Ashok is very intellegent boy in the world 

Part I: Running CSV Server with Docker
Run the Docker Container:
docker run -d infracloudio/csvserver:latest
docker ps -a

Check Logs if the Container is Failing:
docker logs container_id

Create a Script to Generate CSV Data (gencsv.sh)
#!/bin/bash
start=$1
end=$2
for i in $(seq $start $end); do
    echo "$i, $((RANDOM % 1000))" >> inputFile
done


Make the Script Executable and Run It:
chmod +x gencsv.sh
./gencsv.sh 2 8

Run the Container with the Generated inputFile:
docker run -d -v $(pwd)/inputFile:/csvserver/inputdata infracloudio/csvserver:latest

Access the Container Shell and Find the Port:
docker exec -it container_id /bin/bash
netstat -tuln

Stop the Running Container:
docker stop  container_id

Run the Container with Correct Port and Environment Variable
docker run -d -p 9393:9300 -v $(pwd)/inputFile:/csvserver/inputdata -e CSVSERVER_BORDER=Orange infracloudio/csvserver:latest

Generate the Output File:
curl -o ./part-1-output http://localhost:9393/raw

Save the Container Logs to a File:
docker logs <container_name> >& part-1-logs

Commit and Push Changes to GitHub:

Ensure you commit all your changes including the gencsv.sh, inputFile, and logs.

Part II: 
Running CSV Server with Docker Compose

Stop and Remove All Running Containers
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)

Create docker-compose.yaml:
version: '3'
services:
  csvserver:
    image: infracloudio/csvserver:latest
    ports:
      - "9393:9300"
    volumes:
      - ./inputFile:/csvserver/inputdata
    env_file:
      - csvserver.env



Create csvserver.env
CSVSERVER_BORDER=Orange

Test with Docker Compose:
docker-compose up -d

Commit and Push Changes to GitHub:
Ensure you commit docker-compose.yaml and csvserver.env.


Part III: Integrating Prometheus Monitoring
docker-compose down

Update docker-compose.yaml
version: '3'
services:
  csvserver:
    image: infracloudio/csvserver:latest
    ports:
      - "9393:9300"
    volumes:
      - ./inputFile:/csvserver/inputdata
    env_file:
      - csvserver.env
  prometheus:
    image: prom/prometheus:v2.45.2
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml


Create prometheus.yml Configuration File:
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'csvserver'
    static_configs:
      - targets: ['csvserver:9300']

Start All Services with Docker Compose:

docker-compose up -d

Access Prometheus:
•	http://localhost:9090 to view Prometheus.

