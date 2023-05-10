![](.Readme_images/a028d5bc.png)

# setup grafana
masuk ke dashboard grafana

masukan username admin, dan password admin 
![](.Readme_images/4196c61f.png)

ubah password 

![](.Readme_images/e2c9720c.png)

masuk ke setting > configuration klik add datasource 

![](.Readme_images/cca1ab64.png)

pilih prometheus

masukan url, edit form sesuai kebutuhan, klik klik save & test 

![](.Readme_images/c12386fb.png)

# setup dashboard grafana
klik ikon kotak, pilih new dashboard

![](.Readme_images/39b3c072.png)

pilih add a new panel

klik code, masukan rumus query, untuk penggunaan CPU
```shell
100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle", instance="IP:9100"}[5m])) * 100 ) 
```

untuk penggunaan memory
```yaml
100 * ((node_memory_MemTotal_bytes{instance="10.116.106.150:9100"} - node_memory_MemFree_bytes{instance="10.116.106.150:9100"} - node_memory_Buffers_bytes{instance="10.116.106.150:9100"} - node_memory_Cached_bytes{instance="10.116.106.150:9100"}) / node_memory_MemTotal_bytes{instance="10.116.106.150:9100"})
```

untuk free disk space
```shell
node_filesystem_avail_bytes{instance="10.116.106.170:9100"} / 1073741824
```

untuk jenkins success build
```shell
sum(default_jenkins_builds_success_build_count_total)
```

untuk jenkins failed build
```shell
sum(default_jenkins_builds_failed_build_count_total)
```
![](.Readme_images/68213d70.png)    

save dashboard 
![](.Readme_images/2ae9dec3.png)
![](.Readme_images/d704db9c.png)

# alerting

## setup integrasi dengan slack dengan grafana

masuk ke https://api.slack.com , pilih create an app 
![](.Readme_images/66d2fe67.png)

from stach

![](.Readme_images/00a77614.png)

masukan app name, pilih workspace 

![](.Readme_images/5bc104b2.png)

pilih incoming webhook 

![](.Readme_images/890e0c2c.png)

activate incoming webhook 

![](.Readme_images/471f35bd.png)

add new webhook to workspace 

![](.Readme_images/626c5851.png)

pilih workspace slack 

![](.Readme_images/1bf31d65.png)

copy webhook url 

![](.Readme_images/066f0178.png)

pada dashboard grafana, klik alerting, notification policies

![](.Readme_images/ecfe1174.png)

pada root policies, pilih edit > create a contact point masukan nama, contact point types, webhook url, klik test 

![](.Readme_images/9cef020d.png)

integrasi berhasil 

![](.Readme_images/98d852cd.png)

save contact point 

![](.Readme_images/c8dd7fc4.png)

ubah default for all alert ke slack 
![](.Readme_images/d264aebf.png)

## membuat alert

edit panel, pilih alert, create alert rule from this panel

![](.Readme_images/1994e028.png)
![](.Readme_images/3b7a98a8.png)

save and quit

setup alert berhasil 

![](.Readme_images/3a5780e2.png)