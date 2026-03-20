## Mosquitto

[Eclipse Mosquitto](https://mosquitto.org) is an open source message broker which implements a server for MQTT. It runs in Docker and is exposed on the default MQTT port `1883`. You can subscribe to and push into `topics`: 

```sh
mosquitto_sub -h localhost -t '#' -p 1883
mosquitto_pub -h localhost -p 1883 -t '/' -m $(date --utc +%s)
```

A useful tool to debug MQTT is  [MQTT Explorer](https://mqtt-explorer.com/) found on [Github](https://github.com/thomasnordquist/MQTT-Explorer/).

## Node-RED

[Node-RED](https://nodered.org) is a low-code programming tool for wiring together hardware devices, APIs and online services in new and interesting ways.

It provides a browser-based editor that makes it easy to wire together flows using the wide range of nodes in the palette that can be deployed to its runtime in a single-click.

Node-RED is also running in Docker and is exposed on port `1880`: http://localhost:1880/

## InfluxDB

[InfluxDB](https://www.influxdata.com) is a database for any time series data. It runs in Docker and is exposed on port `8086`: http://localhost:8086/ (you have to create an initial user in just a few simple steps)

## Grafana

[Grafana](https://grafana.com) is visualisation software for building operational dashboards. It runs in Docker and is exposed on port `3000`:

You can login to Grafana: http://localhost:3000/login (admin:admin)

## Jupyter Notebook

[Jupyter Notebook](https://jupyter.org/) is a web application for creating and sharing computational documents.
It runs in Docker and is exposed on port `8080`: http://localhost:8080/ (token: laudsgateway)

## FastAPI

[FastAPI](https://fastapi.tiangolo.com/) is a modern Python web framework for building high-performance APIs.
It runs in Docker and is exposed on port `9000`: http://localhost:9000/

## Interfacer API

[Interfacer API](https://github.com/interfacerproject/Interfacer-notebook) is an API service for interacting with the [Interfacer Project](https://interfacerproject.dyne.org/)

## Frontend

The Frontend is a web-based user interface for interacting with the system and visualizing engergy analytics.
It runs in Docker using a web server and is exposed on port 8088: http://localhost:8088/