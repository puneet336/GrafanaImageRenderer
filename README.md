# GrafanaImageRenderer
I had downloaded and extracted the grafana tarball at /root/grafana-9.3.2. 

1. Download the grafana image renderer plugin 
```
[root@web01 grafana-9.3.2]# cd /root/grafana-9.3.2
[root@web01 grafana-9.3.2]# bin/grafana-cli plugins install grafana-image-renderer
✔ Downloaded and extracted grafana-image-renderer v3.6.4 zip successfully to /var/lib/grafana/plugins/grafana-image-renderer
```


2. this will generally install  plugins at /var/lib/grafana/plugins. Here's my grafana.ini - 

```
#################################### Paths ###############################
[paths]
# Path to where grafana can store temp files, sessions, and the sqlite3 db (if that is used)
data = data

# Temporary files in `data` directory older than given duration will be removed
temp_data_lifetime = 24h

# Directory where grafana can store logs
logs = data/log

# Directory where grafana will automatically scan and look for plugins
plugins = data/plugins

# folder that contains provisioning config files that grafana will apply on startup and while running.
provisioning = conf/provisioning

#################################### Server ##############################
[server]
```
as plugins=data/plugins so i had to - 
[root@web01 grafana-9.3.2]# cd data
[root@web01 data]# ln -sf /var/lib/grafana/plugins .
[root@web01 grafana-9.3.2]# cd ..

3. Now start the grafana server service.
```
[root@web01 grafana-9.3.2]# bin/grafana-server
```
and notice the logs -
```
Grafana server is running with elevated privileges. This is not recommended
INFO [02-27|10:50:33] Starting Grafana                         logger=settings version=9.3.2 commit=21c1d14e91 branch=HEAD compiled=2022-12-14T16:10:18+05:30
INFO [02-27|10:50:33] Config loaded from                       logger=settings file=/root/grafana-9.3.2/conf/defaults.ini
INFO [02-27|10:50:33] Path Home                                logger=settings path=/root/grafana-9.3.2
INFO [02-27|10:50:33] Path Data                                logger=settings path=/root/grafana-9.3.2/data
INFO [02-27|10:50:33] Path Logs                                logger=settings path=/root/grafana-9.3.2/data/log
INFO [02-27|10:50:33] Path Plugins                             logger=settings path=/root/grafana-9.3.2/data/plugins
INFO [02-27|10:50:33] Path Provisioning                        logger=settings path=/root/grafana-9.3.2/conf/provisioning
INFO [02-27|10:50:33] App mode production                      logger=settings
INFO [02-27|10:50:33] Connecting to DB                         logger=sqlstore dbtype=sqlite3
INFO [02-27|10:50:33] Starting DB migrations                   logger=migrator
INFO [02-27|10:50:33] migrations completed                     logger=migrator performed=0 skipped=464 duration=369.896µs
INFO [02-27|10:50:33] Plugin registered                        logger=plugin.loader pluginID=input
INFO [02-27|10:50:37] Plugin registered                        logger=plugin.loader pluginID=grafana-image-renderer
INFO [02-27|10:50:37] Envelope encryption state                logger=secrets enabled=true current provider=secretKey.v1
INFO [02-27|10:50:37] Query Service initialization             logger=query_data
INFO [02-27|10:50:37] Live Push Gateway initialization         logger=live.push_http
INFO [02-27|10:50:37] registering usage stat providers         logger=infra.usagestats.collector usageStatsProvidersLen=2
INFO [02-27|10:50:37] starting to provision alerting           logger=provisioning.alerting
INFO [02-27|10:50:37] finished to provision alerting           logger=provisioning.alerting
INFO [02-27|10:50:37] HTTP Server Listen                       logger=http.server address=[::]:3000 protocol=http subUrl= socket=
INFO [02-27|10:50:37] Warming state cache for startup          logger=ngalert.state.manager
INFO [02-27|10:50:37] storage starting                         logger=grafanaStorageLogger
INFO [02-27|10:50:37] State cache has been initialized         logger=ngalert.state.manager states=0 duration=42.083393ms
INFO [02-27|10:50:37] starting                                 logger=ticker first_tick=2023-02-27T10:50:40+05:30
INFO [02-27|10:50:37] starting MultiOrg Alertmanager           logger=ngalert.multiorg.alertmanager
WARN [02-27|10:50:38] Plugin process is running with elevated privileges. This is not recommended logger=plugin.grafana-image-renderer
INFO [02-27|10:50:38] Request Completed                        logger=context userId=1 orgId=1 uname=admin method=GET path=/api/live/ws status=-1 remote_addr=192.168.29.56 time_ms=4 duration=4.798244ms size=0 referer= handler=/api/live/ws
INFO [02-27|10:50:38] Initialized channel handler              logger=live channel=grafana/dashboard/uid/ortu5FxVz address=grafana/dashboard/uid/ortu5FxVz
INFO [02-27|10:50:50] Request Completed                        logger=context userId=1 orgId=1 uname=admin method=GET path=/api/live/ws status=-1 remote_addr=192.168.29.56 time_ms=1 duration=1.722581ms size=0 referer= handler=/api/live/ws
```


4. now that image renderplugin is registered and plugin process is reported to be running, get a authorization token - 
```
curl -X POST -H "Content-Type: application/json" -d '{"name":"apikeycurl", "role": "Admin"}' http://admin:admin@localhost:3000/api/auth/keys
{"id":1,"name":"apikeycurl","key":"eyJrIjoidXZWZk1jTXdYTjFXMkQxTTBxbWVmWWtQS3I2UHBJZFEiLCIjoiYXBpa2V5Y3VybCIsImlkIjoxfQ=="}
```
if you get following response, then you need to change "name" from apikeycurl to apikeycurl1   Or,

delete the existing key from settings menu.
<img width="180" alt="image" src="https://user-images.githubusercontent.com/5935825/221489515-ad3ffcb7-ab9c-4eca-9e13-ee762d9f6aab.png">

5. Now that  you have the API Keys, get the dashboard url as - 
![image](https://user-images.githubusercontent.com/5935825/221490436-cba9fd03-ee83-4b5d-88fa-f6f944b03af7.png)
<img width="537" alt="image" src="https://user-images.githubusercontent.com/5935825/221490906-19b287e5-f147-4ca9-bce4-1b89608ddf92.png">


6. use the API Key /Bearer token and renderer url as - 
curl -o 4.png -H "Authorization: Bearer eyJrIjoidXZWZk1jTXdYTjFXMkQxTTBxbWVmWWtQS3I2UHBJZFEiLCIjoiYXBpa2V5Y3VybCIsImlkIjoxfQ==" "http://192.168.29.128:3000/render/d-solo/ortu5FxVz/memorydashboard?orgId=1&from=1677454413037&to=1677476013037&panelId=2&width=1000&height=500"



Thanks all folks!
