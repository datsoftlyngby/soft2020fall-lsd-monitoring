# Your Turn - Scenario


![](images/your_turn.gif)

Reproduce the following scenario to get a first feeling for how to work with Prometheus and Grafana.

This scenario is described on a Linux machine. It will likely not work with a Native Docker installation on Windows and OS X. You can find the entire source code for this scenario on Github https://github.com/datsoftlyngby/soft2020fall-lsd-monitoring


## Starting a web-application

This web-application does nothing more that displaying a welcome page on the root endpoint, i.e. `http://<your_host>:8080/`. However, it serves a set of Prometheus metrics on the `http://<your_host>:8080/metrics`

Run the web-application:
~~~bash
$ cd my_app
$ docker build -t promserver .
$ docker run --rm --name appserver -p 8080:8080 -d promserver
...
$ docker stop <first hex digits of id>
~~~

While starting the webservice inspect the code of the web-application. In particular have a look at how the metrics are logged via the Prometheus client library.

Navigate to https://prometheus.io/docs/instrumenting/clientlibs/ to find a suitable client library for your project.

Observe that the server application is really running. Navigate to `http://<your_host>:8080`. You should see a page saying:

  > **Welcome**
  >
  > To this server.

## Starting a Client,

The client program simulates user interactions on the server by sending one request per second to get the main page.

~~~bash
$ docker run --rm --name appclient -it promclient
~~~

Now, navigate your browser to `http://<your_host>:8080/metrics` and refresh from time to time to see the metrics changing



## Starting Prometheus to Collect Metrics

Now, we let Prometheus collect our applications metrics:

~~~bash
$ docker run --name prometheus --net="host" -v `pwd`/prometheus.yml:/etc/prometheus/prometheus.yml -p 9090:9090 -d prom/prometheus
~~~

Navigate your browser to `http://<your_host>:9090/graph?g0.range_input=1h&g0.expr=cpu_load_percent&g0.tab=0&g1.range_input=1h&g1.expr=http_responses_total&g1.tab=0&g2.range_input=1h&g2.expr=request_duration_milliseconds_bucket&g2.tab=0` to see your metrics through Prometheus inbuilt dashboard.


## Starting Grafana and Instantiating a Dashboard

Finally, to get visually more appealing metrics and to configure alarms, we start up Grafana

~~~bash
$ docker run --net="host" --name grafana -p 3000:3000 -d grafana/grafana:4.5.2
~~~


Navigate your browser to `http://<your_host>:3000` and login with the default credentials `admin`/`admin`. Remember later to change the password for your projects!

Now do the following:

  * `Add data source`
  * Set the `Name` to `My App`
  * Under `Config` choose the `Type` `Prometheus`
  * Under `Http settings` set the `Url` to `http://<your_host>:9090`
  * Finally, click `Add`

Now, `Create your first dashboard`, add a `Graph` click on the `Panel Title` and choose `Edit`.
Set `Data Source` to `My App` and add the PromQL query  `http_responses_total{group="production",instance="localhost:8080",job="example-my-app"}` to the field below.

Now, play and customize the dashboard a bit to your liking and add some other metrics.


### Installing new Panels

In case you need another panel type for example a gauge for the CPU load and in case you are running Grafana via Docker follow the steps below.

  * Navigate to https://grafana.com/plugins?type=panel
  * Choose a panel of your liking, e.g., https://grafana.com/plugins/briangann-gauge-panel/installation
  * Copy the installation command, which has to be run on the Grafana server (machine)

~~~bash
vagrant@vagrant:~$ docker exec -it lsd_monitoring_grafana_1 /bin/bash
root@vagrant:/# grafana-cli plugins install briangann-gauge-panel
installing briangann-gauge-panel @ 0.0.6
from url: https://grafana.com/api/plugins/briangann-gauge-panel/versions/0.0.6/download
into: /var/lib/grafana/plugins

✔ Installed briangann-gauge-panel successfully

Restart grafana after installing plugins . <service grafana-server restart>

root@vagrant:/# exit
exit
vagrant@vagrant:~$ docker restart lsd_monitoring_grafana_1
~~~
