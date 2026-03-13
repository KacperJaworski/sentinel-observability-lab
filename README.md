# PROJECT NOTES Project Sentinel: Full-Stack Observability & Resilience Lab

STACK:
Vmware Workstation Pro | Ubuntu | Ansible | Docker | Nginx | Bombardier | Filebeat | ELK ElasticSearch + Kibana | Prometheus + NodeExporter + cAdvisor + Alert Manager | Grafana

VMware Workstation Pro: Program do wirtualizacji, pozwalajacy na postawienie roznych OS na jednym komputerze
Ubuntu Server: Czysty system operacyjny Linux
Ansible: Narzędzie do automatyzacji - loguje się na serwer i instaluje wszystko za pomocą kodu - mowi dockerowi co ma postawic
Docker: Silnik, który uruchamia i utrzymuje aplikacje w izolowanych kontenerach

Nginx: Serwer WWW (ofiara) – hostuje stronę i generuje logi tekstowe z wejść oraz błędów
Bombardier: Generator ruchu – masowo wysyła zapytania HTTP do Nginxa, by maksymalnie go obciążyć.

Filebeat: Kurier siedzacy kolo Nginxa – na bieżąco czyta pliki tekstowe z logami Nginxa i wysyła je do bazy
Elasticsearch: Potężna baza danych – magazynuje wszystkie logi tekstowe zebrane przez Filebeata
Kibana: Panel graficzny do bazy Elasticsearch – tu przeglądasz logi i szukasz tekstowych przyczyn awarii.

Prometheus: Zbieracz danych liczbowych – cyklicznie pobiera i zapisuje stan sprzętu (użycie CPU, RAM, czas odpowiedzi).
Node Exporter & cAdvisor: Agenci sprzętowi – dostarczają Prometheusowi dane o obciążeniu całego serwera i samych kontenerów.
Alertmanager: System alarmowy – wysyła powiadomienie do użytkownika, gdy w Prometheusie przekroczono krytyczny próg (np. brak RAMu).
Grafana: Ostateczne centrum dowodzenia – łączy surowe liczby z Prometheusa i logi w piękne, czytelne dashboardy na jednym ekranie.

Na wirtualnej maszynie z Ubuntu używam skryptów Ansible do automatycznego wdrożenia Dockera i uruchomienia wszystkich systemów w odzielnych kontenerach. Następnie Bombardier sztucznie przeciąża serwer Nginx, generując awarie, których logi tekstowe na bieżąco zbiera i analizuje stack ELK. Elasticsearch jako baza pobiera dane wysylane przez filebeat, a kibana te logi wyswietla. Równocześnie Prometheus mierzy krytyczne zużycie sprzętu (CPU/RAM) za pomoca node exportera i cadvisora, a wszystkie te dane łączy i wyświetla na jednym ekranie Grafana. Alerty trafiaja za pomoca alertmanagera do uzytkownika.

- Install Vmware Workstation Pro 25H2
- Download Ubuntu Server 24.04.4 LTS
- Change IP to Static by update netplan in files: (192.168.225.128/24 - VM)
- In fluent terminal - connect ssh by kacper@192.168.225.128
- Install Ansible in fluent
  sudo apt update | sudo apt install ansible -y | ansible version 2.16.3
- Open VisualStudio install remote-ssh extention
  connect to shh in VS
- Create new folder by mkdir project-sentinel in VS
- Create new yaml file to install docker: 🚀
---
- name: Install Docker on the server
  hosts: localhost
  become: yes
  tasks:
    - name: Install Docker package
      apt:
        name: docker.io
        state: present
        update_cache: yes
    - name: Ensure Docker service is running
      service:
        name: docker
        state: started
        enabled: yes 🚀
      
-  ansible-playbook install-docker.yml -K -> to install docker by yaml file
- Create new yaml file to install nginx by docker compose: 🚀
    services:
    nginx:
      image: nginx:latest
      container_name: sentinel-nginx
      ports:
        - "80:80"
      restart: unless-stopped 🚀
- Now our site Nginx is running - ip the same as VM http://192.168.225.128
- We are increasin mapping limit on linux by: sudo sysctl -w vm.max_map_count=262144
- Install ElasticSearch by adding to docker-compose.yml: 🚀
    elasticsearch:
    image: elasticsearch:8.12.2
    container_name: sentinel-elastic
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms1g -Xmx1g
    ports:
      - "9200:9200"
    restart: unless-stopped 🚀
                                next: docker compose up -d
- Now we got 2 containers in Docker: nginx and elasticsearch
- Updating docker-compose.yml by adding kibana depends on elasticsearch: 🚀
    kibana:
    image: kibana:8.12.2
    container_name: sentinel-kibana
    environment:
      - ELASTICSEARCH_HOSTS=http.//elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    restart: unless-stopped 🚀
- kibana - http://192.168.225.128:5601
- Now we got 3 containers: Nginx, Elasticsearch and Kibana
- We have to add new yaml file to configure filebeat -> to take logs from Nginx and upload them to Elasticsearch
- filebeeat.yml: 🚀
  filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/auth.log
    - /var/log/syslog

output.elasticsearch:
  hosts: ["elasticsearch:9200"]

setup.kibana:
  host: "kibana:5601" 🚀
- We have to change file owner to root by: sudo chown root filebeat.yml and sudo chmod 644 filebeat.yml
- Now our kibana is working, and we can create dashboards
- We are creating new file: prometheus.yml -> in this file, we are configuring prometheus, node-exporter to count procesor using and RAM on whole ubuntu server, and cadvisor to count which container using memory. prometheus.yml: 🚀
  global:
    scrape_interval: 15s

  scrape_configs:
    - job_name: 'prometheus'
      static_configs:
        - targets: ['localhost:9090']
  
    - job_name: 'node-exporter'
      static_configs:
        -targets: ['node-exporter:9100']

    - job_name: 'cadvisor'
      static_configs:
        - targets: ['cadvisor:8080'] 🚀
- Then, we have to modificate docker-compose.yml file: 🚀
    prometheus:
      image: prom/prometheus:latest
      container_name: sentinel-prometheus
      volumes:
        - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      ports:
        - "9090:9090"
      restart: unless-stopped

    node-exporter:
      image: prom/node-exporter:latest
      container_name: sentinel-node-exporter
      ports:
        - "9100:9100"
      restart: unless-stopped

    cadvisor:
      image: gcr.io/cadvisor/cadvisor:v0.47.0
      container_name: sentinel-cadvisor
      volumes:
        - /:/rootfs:ro
        - /var/run:/var/run:ro
        - /sys:/sys:ro
        - /var/lib/docker/:/var/lib/docker:ro
        - /dev/disk/:/dev/disk:ro
      ports:
        - "8080:8080"
      privileged: true 
      restart: unless-stopped 🚀
- Prometheus: http://192.168.225.128:9090/
- We have to configure grafana in docker-compose.yml by:
    grafana:
    image: grafana/grafana:latest
    container_name: sentinel-grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
    restart: unless-stopped 🚀
- Grafana is now working: http://192.168.225.128:3000/ -> login: admin password - the same as vm
- We have to add data source in grafana by writing prometheus URL: http://prometheus:9090
- We took already exists ID of dashboard to visualize node exporter full [ id = 1860], and cadvisor [id = 14282] and configure them to visualize our containers data, and ubuntu whole server data memory
- We are adding AlertManager to alert as if CPU usage > 80%, we have to add file alerts.yml: 🚀
  groups:
  - name: sentinel_alerts
    rules:
      - alert: HighCpuUsage
        expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100) > 80
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "Critical CPU usage detected!"
          description: "Warning! CPU usage has exceeded 80% for the last 30 seconds. Possible stress test or attack in progress!" 🚀
- Then alertmanager.yml: 🚀
  route:
    receiver: 'default-receiver'

  receivers:
    - name: 'default-receiver' 🚀
- Edit file docker-compose.yml by adding alert manager, and then configure volumes of prometheus: 🚀
    alertmanager:
    image: prom/alertmanager:latest
    container_name: sentinel-alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml/:/etc/alertmanager/alertmanager.yml:ro
    restart: unless-stopped
    depends_on:
      - prometheus
  and
    in prom section:  - ./alerts.yml:/etc/prometheus/alerts.yml:ro 🚀
  - Our alertmanager is working on: http://192.168.225.128:9093/#/alerts
  - We have our alert on prometheus site in section alerts:
    <img width="1903" height="413" alt="image" src="https://github.com/user-attachments/assets/8bd2b267-e565-4b70-8217-ab05938db755" />

  
