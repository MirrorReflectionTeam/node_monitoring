# monitoring-tool

### You need:
- Server with node
- Server for monitoring tool: Ubuntu 20.04 / 1 VCPU / 2 GB RAM / 20 GB SSD

## Install node exporter on validator node
Just run next command 
```
bash <(curl https://raw.githubusercontent.com/MirrorReflectionTeam/node_monitoring/main/utils/install_exporter.sh)
```

Make sure prometheus is enabled in validator **config.toml** file
##### Open ports if you use firewall
```
ufw allow 9100
ufw allow <prometheus_port_node>
```

## Install monitoring stack on your server for monitoring

### Install docker
```
bash <(curl -s https://raw.githubusercontent.com/MirrorReflectionTeam/node_monitoring/main/utils/docker_install.sh)
```

### Clone the repository
```
cd ~
git clone https://github.com/MirrorReflectionTeam/node_monitoring.git
cd node_monitoring 
```

### Configuration:
#### Server for monitoring:
Add your servers with installed [node_exporter](https://github.com/prometheus/node_exporter) or installed cosmos-based node with enabled prometheus port to file <b>prometheus/prometheus.yml</b>
```
  ##############################
  ##     just remove '#'      ##
  ##  and replace on your ip  ##
  ##############################

  ## for monitoring your servers
  #- job_name: "your_servers"
  #  static_configs:
  #  - targets: ["127.0.0.1:9100"]
  #    labels:
  #      instance: "server1"
  #  - targets: ["127.0.0.2:9100"]
  #    labels:
  #      instance: "server2"


  #################################
  ##       just remove '#'       ##
  ##    and replace on your ip   ##
  ## and on your prometheus port ##
  #################################

  ## for cosmos-based validator node with node_exporter installed and prometheus enabled
  #- job_name: "cosmos-validator"
  #  static_configs:
  #  - targets: ["127.0.0.1:9100","127.0.0.1:26660"]
  #    labels:
  #      instance: "node1"
  #  - targets: ["127.0.0.1:9100","127.0.0.1:27660"]
  #    labels:
  #      instance: "node2"
  #  - targets: ["127.0.0.1:9100","127.0.0.1:28660"]
  #    labels:
  #      instance: "node3"
```

### Alerts:
#### Server alerts
- Server down
- Out of memory (<10%)
- Out of disk space (<10%)
- Out of disk space within 24h
- High CPU load (>85%)

#### Cosmos-based validator node alerts
- Missing blocks
- Degraded syncing (sync less than 40 blocks in last 5 min)
- Low peers count (< 3)
- Node time out of Sync
- Node Disk Full


#### Telegram notification:
| KEYS	| VALUES |
| :------: | :-----:
| TELEGRAM_ADMIN |	Your **chat id** you can get from [this](@userinfobot) |
|TELEGRAM_TOKEN	| Your telegram **bot token** you can get from [this](@botfather)|

In order to enable telegram notifications, create your own bot and fill in the following fields in the file <b>alertmanager/config.yml</b>
```
chat_id=1111111                 
bot_token=11111111:AAG_XXXXXXX  
```


#### Start containers
```
sudo docker compose up -d
```

##### Open in browser http://<your_server_ip>:8888 <br>
##### Default credentials: admin/admin



#### SSL
If you would like to set up SSL then use the following instruction 
Open docker-compose file in text editor and uncomment a part with Caddy service
```
  reverse-proxy:
    container_name: caddy
    image: caddy/caddy:2-alpine
    restart: unless-stopped
    volumes:
      - caddy_data:/data
      - caddy_config:/config
      - ./Caddyfile:/etc/caddy/Caddyfile
    ports:
      - 80:80
      - 443:443
    networks:
      - monitoring
```
##### Second step - replace `example.domain.com` with your domain name in Caddyfile:
```
example.domain.com {
	reverse_proxy grafana:8888
}
```

