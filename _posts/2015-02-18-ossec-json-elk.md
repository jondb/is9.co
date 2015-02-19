---
layout: post
title: OSSEC JSONOUT to ELK
---

Logstash and OSSEC -- never easier.

ELK and OSSEC are now setup to write alerts to alerts.json, and you can use lumberjack and the json codec to feed those logs to the logstash service. Here's how:

    git clone https://github.com/ossec/ossec-hids.git

    cd ossec-hids
    sudo ./install.sh

Follow the prompts as usual. Choose update. Or choose server if a fresh install.

Once install is finished, add the jsonout_output = yes to the config at:

    /var/ossec/etc/ossec.conf

Update the global section to include:

    <jsonout_output>yes</jsonout_output>

Restart OSSEC:

    /var/ossec/bin/ossec-control restart

Verify the new json file is in place:

    cat /var/ossec/logs/alerts/alerts.json


Setup logstash-forwarder. Install as normal, then build your config at /etc/logstash-forwarder:

    {
      "network": {
        "servers": [ "<logstash server>:<port>" ],
        "timeout": 15,
        "ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt"
      },
      "files": [
        {
          "paths": [
              "/var/ossec/logs/alerts/alerts.json"
           ],
          "fields": { "type": "ossec-alerts" }
        }
       ]
    }

Restart with:

    /etc/init.d/logstash-forwarder restart

Setup logstash to handle json files. Go to you logstash server. I assume you have installed logstash, elastic search and kibana. This is /etc/logstash/conf.d/10-default.conf on my setup.

    input {
      lumberjack {
        port => 10516
        type => "lumberjack"
        ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
        ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
        codec => json
      }
    }

    output {
      elasticsearch { host => localhost }
    }


