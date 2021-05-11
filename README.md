# Telegraf deploy to K3S

File `telegraf-helm-values.yaml` has values for official telegraf deployment and directory `telegraf.d` contains configuration
files to be mounted at `/etc/telegraf/telegraf.d`. Also, it is necessary to have configuration for environment variables.

## Enviroment configuration

Add helm repo:

`helm repo add influxdata https://helm.influxdata.com/`

#### Create a secret to hold credentials as enviroment variables:

`kubectl -n influxdb-server create secret generic telegraf-credentials-env --from-file=influxv1-username=./influxv1-username.txt --from-file=influxv1-password=./influxv1-password.txt --from-file=influx-token=./influx-token.txt`

Each `.txt` file above shall have only a single line with user, password and token respectively. Attention: do not leave a blank line after 
the secret in the first line, not a single space. Everything will be part of the secret and will cause problems.

#### Create a  configmap to hold plugin configurations (input and outputs `.conf` files)

`kubectl -n influxdb-server create configmap telegraf-out-influx --from-file=./conf.d/`

#### Use helm to install the chart

`helm -n influxdb-server upgrade --install telegraf -f telegraf-helm-values.yaml influxdata/telegraf`

And follow helm instruction to see logs of the pod. If everything was right, logs shall be similar to:

```
2021-05-11T03:30:01Z I! Starting Telegraf 1.18.2
2021-05-11T03:30:01Z I! Using config file: /etc/telegraf/telegraf.conf
2021-05-11T03:30:01Z I! Loaded inputs: internal (2x) modbus snmp
2021-05-11T03:30:01Z I! Loaded aggregators: 
2021-05-11T03:30:01Z I! Loaded processors: enum (3x)
2021-05-11T03:30:01Z I! Loaded outputs: influxdb (2x) influxdb_v2 (2x) mqtt
2021-05-11T03:30:01Z I! Tags enabled: 
2021-05-11T03:30:01Z I! [agent] Config: Interval:10s, Quiet:false, Hostname:"telegraf-polling-service", Flush Interval:10s
```