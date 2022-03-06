# Pi-Hole-Grafana-Prometheus-Influxdb-Telegraf-unbound

IMP Docs and Links

**Unbound**
************************************************************************************************************************************************************************************************************************
**Configuring Unbound as a simple forwarding DNS server  (linux VM)
https://www.redhat.com/sysadmin/forwarding-dns-2

**Configuring Unbound Docker 

docker run --detach=true \
--name=my-unbound \
--volume=/volume1/docker/unbound:/opt/unbound/etc/unbound/ \
--cap-add NET_ADMIN \
--publish=53:53/tcp \
--publish=53:53/udp \
--env "ServerIP=192.168.0.4" \
--env "DNSMASQ_LISTENING=all \
--ip "192.168.0.4" \
mvance/unbound:latest


unbound.conf
server:
do-ip6: no
cache-max-ttl: 14400
cache-min-ttl: 300
hide-identity: yes
hide-version: yes
minimal-responses: yes
prefetch: yes
qname-minimisation: yes
rrset-roundrobin: yes
use-caps-for-id: yes
username: "_unbound"
aggressive-nsec: yes
do-not-query-localhost: no
neg-cache-size: 4M
qname-minimisation: yes
deny-any: yes
harden-algo-downgrade: yes
harden-below-nxdomain: yes
harden-dnssec-stripped: yes
harden-glue: yes
harden-large-queries: yes
harden-referral-path: no
harden-short-bufsize: yes
hide-http-user-agent: no
hide-identity: yes
hide-version: yes
http-user-agent: "DNS"
identity: "DNS"
private-address: 10.0.0.0/8
private-address: 172.16.0.0/12
private-address: 192.168.0.0/16
private-address: 169.254.0.0/16
ratelimit: 1000
unwanted-reply-threshold: 10000
use-caps-for-id: yes
val-clean-additional: yes
tls-cert-bundle: /etc/ssl/certs/ca-certificates.crt
infra-cache-slabs: 4
incoming-num-tcp: 10
key-cache-slabs: 4
msg-cache-size: 1591685802
msg-cache-slabs: 4
num-queries-per-thread: 4096
num-threads: 3
outgoing-range: 8192
rrset-cache-size: 3183371605
rrset-cache-slabs: 4
minimal-responses: yes
prefetch: yes
prefetch-key: yes
serve-expired: yes
so-reuseport: yes

# If no logfile is specified, syslog is used
logfile: "/opt/unbound/etc/unbound/unbound.log"
verbosity: 4
interface: 0.0.0.0
interface: ::0
Access-control: 127.0.0.0/8 allow  # (allow queries from the local host)
access-control: 192.168.0.0/24 allow
access-control: 0.0.0.0/24 allow
access-control: 192.168.0.0/24 allow
domain-insecure: "forest.local"
#file to read root hints from.
root-hints: "/opt/unbound/etc/unbound/root.hints"
port: 53
forward-zone:
      name: "."
      forward-addr: 1.1.1.1
      forward-addr: 8.8.8.8

********************************************************************************************************************************************************************************
**Install and Start Pi-Hole**

MacVlan (refer https://github.com/chanakya-1984/techtalk/tree/master/Docker_series/07)

docker network create --driver=macvlan --gateway=192.168.0.x --subnet=192.168.0.0/24 --ip-range=192.168.0.254/32 -o parent=ovs_bond2 VLAN1_pi-hole
https://github.com/chanakya-1984/techtalk/tree/master/Docker_series/07


docker run --detach \
           --name pi-hole \
           --restart always \
           --volume /etc/localtime:/etc/localtime:ro \
           --volume /volume1/docker/pi-hole/config:/etc/pihole \
           --volume /volume1/docker/pi-hole/dnsmasq:/etc/dnsmasq.d \
           --cap-add NET_ADMIN \
           --dns=192.168.0.x\
           --env "DNS1=192.168.0.x" \
           --env "ServerIP=192.168.0.x" \
           --env "DNSMASQ_LISTENING=local" \
           --env "WEBPASSWORD=somepassword" \
           --env "TZ=Asia/Dubai" \
           --network VLAN1_pi-hole \
           --ip "192.168.0.x" \
           --hostname "pi-hole" \
           pihole/pihole:latest

*How can i reset all pihole stats?

log into "docker exec -it 9a033050bbxxx bash"

cd /etc/pihole
sudo service pihole-FTL stop
sudo mv pihole-FTL.db pihole-FTL.db.old
sudo service pihole-FTL start

Grafana Pi-hole Dashboards

https://grafana.com/grafana/dashboards/6603

*https://grafana.com/grafana/dashboards/13565

just use telegraf conf and input.http

Sample Telegraf Conf

[[[inputs.http]]
#PiHole URL for data in JSON format
urls = ["http://192.168.0.x/admin/api.php"]

method = "GET"

 #Overwrite measurement name from default `http` to `pihole_stats`
name_override = "pihole_stats"

#Exclude host items from tags
tagexclude = ["host"]

#Data from HTTP in JSON format
data_format = "json"

#JSON values to set as string fields
json_string_fields = ["url", "status"]

insecure_skip_verify = true


********************************************************************************************************************************************************************************

** Install and Start Grafana **

https://grafana.com/docs/grafana/latest/installation/docker/

*Grafana Open Source edition: grafana/grafana-oss:<version>***
  
docker run -d \
  -p 3000:3000 \
  --volume /volume1/docker/unbound:/opt/unbound/etc/unbound/ \
  --name=grafana \
  grafana/latest

docker run --user 104 --volume "<your volume mapping here>" grafana/latest

Modify permissions
The commands below run bash inside the Grafana container with your volume mapped in. This makes it possible to modify the file ownership to match the new container. Always be careful when modifying permissions.

$ docker run -ti --user root --volume "<your volume mapping here>" --entrypoint bash grafana/latest

# in the container you just started:
chown -R root:root /etc/grafana && \
chmod -R a+r /etc/grafana && \
chown -R grafana:grafana /var/lib/grafana && \
chown -R grafana:grafana /usr/share/grafana


**********************************************************************************************************************************************************************************qbittorrent grafana dashboard***

Useful Links

1 https://github.com/fru1tstand/qbittorrent-exporter
1 https://github.com/fru1tstand/qbittorrent-exporter/releases (Download)

2 https://github.com/esanchezm/prometheus-qbittorrent-exporter
2 https://raw.githubusercontent.com/esanchezm/prometheus-qbittorrent-exporter/master/grafana/dashboard.json

**qbittorrent-exporter
Run java -jar <qbt-exporter.jar> [flag].
/usr/bin/java -jar /etc/prometheus/qbt-exporter-fat-1.0.0-release.jar
       
root@xxxxt:/etc/init.d# java -jar qbt-exporter-fat-1.0.0-release.jar
No settings file found, creating one at /etc/init.d/qbt-exporter-settings.json.
[main] INFO org.eclipse.jetty.util.log - Logging initialized @11474ms to org.eclipse.jetty.util.log.Slf4jLog
[main] INFO ktor.application - No ktor.deployment.watch patterns specified, automatic reload is not active

Sample qbt-exporter-settings.json ( change value from False to True)
      
If you want to search and replace the pattern in the entire file, use the percentage character % as a range. This character indicates a range from the first to the last line of the file:

:%s/foo/bar/g
      
 {
  "exporterServerSettings": {
    "port": 9561,
    "host": "0.0.0.0"
  },
  "qbtSettings": {
    "webUiAddress": "http://localhost:8080"
  },
  "collectorSettings": {
    "maindataCollectors": {
      "serverState": {
        "allTimeUploadBytes": false,
        "networkUploadSpeedBytesPerSecond": false,  <- Chang ethis to True
        "allTimeShareRatio": false,
        "diskWriteCacheOverloadPercent": false,


**prometheus-qbittorrent-exporter

pip3 install prometheus-qbittorrent-exporter
qbt-exporter for qbittorrent

Dashboards 

Jason https://raw.githubusercontent.com/esanchezm/prometheus-qbittorrent-exporter/master/grafana/dashboard.json
https://grafana.com/grafana/dashboards/15116
https://raw.githubusercontent.com/esanchezm/prometheus-qbittorrent-exporter/master/grafana/dashboard.json


********************************************************************************************************************************************************************************
**Prometheus-db  - Server Side

Packages

docker run \
    --name Prometheus-db \
    --restart always \
    --publish 9090:9090 \
    --volume /volume1/docker/prometheus:/prometheus-data \
    --volume /volume1/docker/prometheus/etc/prometheus.yml:/etc/prometheus/prometheus.yml \
    --hostname "Prometheus-db" \
    prom/prometheus

Pleae refer prometheus.yml from below

**Prometheus Client Side 

apt install prometheus-node-exporter


http://192.168.0.x:9090/targets
http://192.168.0.x:9090/tsdb-status

********************************************************************************************************************************************************************************
**influxdb2**

*Docker installation
  
docker run -d -p 8086:8086 \
      --name influxdb2 \
      -v /volume1/docker/influxdb:/var/lib/influxdb2 \
      --restart always \
      influxdb:latest

Connect influxdb to grafana using API token (use query language FLUX)

InfluxDB Details
Organization Your InfluxDB organization name or ID.   (create at time of first login)
Token :- Token: Your InfluxDB API token.  ( get it form web ui under data)
Default Bucket: The default bucket to use in Flux queries. 

useful links
http://192.168.0.x:8086/api/v2/authorizations
https://docs.influxdata.com/influxdb/v2.1/tools/grafana/?t=InfluxQL
http://192.168.0.x:8086/api/v2/authorizations
https://docs.influxdata.com/influxdb/v2.1/tools/grafana/?t=InfluxQL
https://docs.influxdata.com/influxdb/v2.1/security/tokens/


**Install Telegraf and configure for InfluxDB

Download the package.
wget https://dl.influxdata.com/telegraf/releases/telegraf_1.20.4-1_amd64.deb
Install it using the package manager.

sudo dpkg -i telegraf_1.20.4-1_amd64.deb
It is normally started by default. We can check its status.

sudo service telegraf status
If it isn't started, then you can start it.

sudo service telegraf start
CD into the new Telegraf folder

cd /etc/telegraf

********************************************************************************************************************************************************************************
sample telegraf.conf

[[outputs.influxdb_v2]]
  urls = ["http://127.0.0.1:8086"]
  token = "YOUR TELEGRAF READ/WRITE TOKEN"
  organization = "YOUR ORG NAME"
  bucket = "telegraf"

[[inputs.cpu]]
  percpu = true
  totalcpu = true
  collect_cpu_time = false
  report_active = false
[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]
[[inputs.diskio]]
[[inputs.mem]]
[[inputs.net]]
[[inputs.processes]]
[[inputs.swap]]
[[inputs.system]]
