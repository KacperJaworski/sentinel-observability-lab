# PROJECT NOTES Project Sentinel: Full-Stack Observability & Resilience Lab

STACK:
Vmware Workstation Pro | Ubuntu | Ansible | Docker | Nginx | Bombardier | Filebeat | ELK ElasticSearch + Kibana + Kibana alert | Prometheus + NodeExporter + cAdvisor + Alert Manager | Grafana

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
