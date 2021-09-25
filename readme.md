# using loki locally for dev log search

## Basic Installation

https://grafana.com/docs/loki/latest/installation/docker/#install-with-docker-compose

```
wget https://raw.githubusercontent.com/grafana/loki/v2.3.0/production/docker-compose.yaml -O docker-compose.yaml
docker-compose -f docker-compose.yaml up
```

## Taking it further

### Wiring it for grafana auto log in

Modify grafana config (don't do the anonymous enabled with admin on non local developer setups):

```
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - ./grafana/provisioning/datasources/:/etc/grafana/provisioning/datasources/
    environment:
      - LOKI_URL=loki
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    networks:
      - loki
```

Create the datasource:

```
mkdir -p grafana/provisioning/datasources
cat <<EOF > grafana/provisioning/datasources/grafana-datasources.json
apiVersion: 1

datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://$LOKI_URL:3100
EOF
```

### Running it against local logs

Modify promtail compose config:

```
  promtail:
    image: grafana/promtail:2.3.0
    volumes:
      - ${LOGS}:/var/log
    command: -config.file=/etc/promtail/config.yml
    networks:
      - loki
```

### Running with changes
Now run it with any folder:

```
LOGS=./logs/examplelog docker-compose -f docker-compose.yaml up
```

## parsing the structured key-value pairs

https://grafana.com/docs/loki/latest/best-practices/

> From early on, we have set a label dynamically using Promtail pipelines for level. This seemed intuitive for us as we often wanted to only show logs for level="error"; however, we are re-evaluating this now as writing a query. {app="loki"} \|= "level=error" is proving to be just as fast for many of our applications as {app="loki",level="error"}.

looks like:

```
{filename="/var/log/example.log"} |= "code = Unknown"
```