# pesho-k/osm-tiles-docker

<!--About-->

This is a **Docker** image that provides a full stack for working w/ **OpenStreetMap** data. It can be used to:

- Initialize **PostgreSQL** database w/ **PostGIS** extensions: `initdb`
- Import **OpenStreetMap** data into **PostgreSQL** database: `import`
- Optionally pre-generate tiles: `render`
- Serve pre-generated (if available) and dynamically generated tiles from **Apache**, **renderd** and **mapnik** via a **Leaflet** interface: `startservices`
- Serve exclusively pre-generated tiles from **Apache** via a **Leaflet** interface: `startweb`
- Update the PostgreSQL database with the latest OSM maps available

## Background

This image was forked from [`zavpyj/osm-tiles-docker`](https://github.com/zavpyj/osm-tiles-docker), which was adapted from [`ncareol/osm-tiles-docker`](https://github.com/ncareol/osm-tiles-docker), which is based on [`homme/openstreetmap-tiles-docker`](https://hub.docker.com/r/homme/openstreetmap-tiles-docker/), which is based on the [Switch2OSM instructions](https://switch2osm.org/serving-tiles/manually-building-a-tile-server-14-04/).

It runs **Ubuntu** 16.04 (Xenial) and is based on [phusion/baseimage-docker](https://github.com/phusion/baseimage-docker). It includes:

- **PostgreSQL** `9.5`
- **PostGIS** extensions
- **Apache** `2.4`
- [**osm2pgsql**](http://wiki.openstreetmap.org/wiki/Osm2pgsql)
- [**mapnik**](http://mapnik.org/)
- [**openstreetmap-carto**](https://github.com/gravitystorm/openstreetmap-carto), a CartoCSS template (mapnik style) for OpenStreetMap data
- [**mod_tile**](http://wiki.openstreetmap.org/wiki/Mod_tile), an **Apache** module that also provides scripts for rendering tiles

## Usage

To build this image:

```sh
$ docker build -t pesho-k/osm-tiles-docker .
```

Command reference is available in `help.txt` or by running the image:

```sh
$ docker run --rm pesho-k/osm-tiles-docker
```

### Build Image
```sh
$ docker-compose build
```

### Atomic Usage

To persist the postgresql database and the generated tiles, it is advised to create beforehand a docker's named volume (mandatory to persist on Windows OS):

```sh
$ docker volume create --name nvpostgisdata -d local
$ docker volume create --name nvtiles -d local
```

Using [`Docker Compose`](https://docs.docker.com/compose/) and a dedicated [`docker-compose.yml`](https://docs.docker.com/compose/compose-file/) configuration file, pesho-k/osm-tiles-docker is even simpler to use:
```sh
$ docker-compose run --rm app-osm initdb
$ docker-compose run --rm app-osm import
$ docker-compose run --rm app-osm render
$ docker-compose up -d
```

### Direct Usage

Initialise if not already done (initdb+import+render) and Start OSM server (startservices)

```sh
$ docker-compose -f osm.yml up -d
```

### Bringing the db in sync with the latest osm map changes

Follow the [`HowTo minutely hstore`](http://wiki.openstreetmap.org/wiki/HowTo_minutely_hstore) guide to produce the necessary **changes.osc.gz** file.
Place the **changes.osc.gz** file in the Docker root directory (same level as the docker-compose files) and run:

```sh
$ docker-compose -f osm-syncdb.yml run --rm app-osm-syncdb syncdb
```
