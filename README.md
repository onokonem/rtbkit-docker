Dokerized RTBkit
===

```shell
sudo docker build -t onokonem/rtbkit https://github.com/onokonem/rtbkit-docker.git

sudo docker run -d --net=host -v /storage/docker/zookeeper/data:/zookeeper-data -p 2181:2181 onokonem/rtbkit-zookeeper

sudo docker run -d --net=host -v /storage/docker/redis/data:/redis-data -p 6379:6379 onokonem/rtbkit-redis

mkdir -vp /storage/docker/graphite/data /storage/docker/graphite/log 
touch /storage/docker/graphite/index

sudo docker run -d --net=host \
  --name graphite \
  -v /storage/docker/graphite/data:/opt/graphite/storage/whisper \
  -v /storage/docker/graphite/index:/opt/graphite/index \
  -v /storage/docker/graphite/log:/var/log \
  -p 8088:8088 \
  -p 8080:8080 \
  -p 2003:2003 \
  -p 8125:8125/udp \
  onokonem/graphite

sudo docker run -d --net=host \
  --name grafana \
  -v /storage/docker/grafana/data:/var/lib/grafana \
  -v /storage/docker/grafana/log:/var/log/grafana \
  -e "GF_SECURITY_ADMIN_PASSWORD=admin"  \
  -p 3000:3000 \
  grafana/grafana

sudo docker run --net=host -t -i \
  -v /storage/docker/rtbkit/log:/var/log \
  -v /storage/docker/rtbkit/log:/opt/rtbkit/logs \
  -e PATH=/opt/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
  -e LD_LIBRARY_PATH=/opt/local/lib: \
  -e PKG_CONFIG_PATH=/opt/local/lib/pkgconfig/:/opt/local/lib/pkg-config/: \
  onokonem/rtbkit /bin/bash

cd /opt/rtbkit && ./build/x86_64/bin/mock_exchange_runner 2> /var/log/mock_exchange_runner.err.log 1> /var/log/mock_exchange_runner.out.log &


tee router-config-openrtb.json <<EOF
/* -*- js-mode -*- */
[
    {
        "exchangeType": "smaato",
        "listenPort": 4444,
        "bindHost": "0.0.0.0",
        "auctionVerb": "POST",
        "auctionResource": "/auctions",
        "performNameLookup": false
    }
]
EOF

sudo docker run --net=host -t -i \
  -v /storage/docker/rtbkit/log:/var/log \
  -v /storage/docker/rtbkit/log:/opt/rtbkit/logs \
  -e PATH=/opt/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
  -e LD_LIBRARY_PATH=/opt/local/lib: \
  -e PKG_CONFIG_PATH=/opt/local/lib/pkgconfig/:/opt/local/lib/pkg-config/: \
  -v $(pwd)/router-config-openrtb.json:/opt/rtbkit/router-config-openrtb.json \
  onokonem/rtbkit /bin/bash
```
