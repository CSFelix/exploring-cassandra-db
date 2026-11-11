# Exploring Cassandra DB

## Summary

* [1. Set up](#1-set-up)
* [2. Contents](#2-contents)

---

## 1. Set up

Run:

```bash
docker compose up -d --build
```

The cluster was designed to be running on a single machine. In order to run in multiple machines, consider updating `Data Center` and `Rack` settings as well as adding more `Cassandra Containers` on [docker-compose.yaml](./docker-compose.yaml).

All keyspaces contain `replication_factor` following the `Network Topology` to all nodes (`3`) and `consistency level` at `QUORUM`.

Every column of type `UUID` will only store `UUIDv7`. This rule is not configurable on database level, we must to apply it on application level.

Oh, and do not forget to unzip `./src/01-use-case-youtube-comments-seed.cql` before running the use case 1 scripts. Since the file is too large for github commit, I needed to compress it.

---

## 2. Contents

* [Use Case 00 - Change Default User](./src/00-use-case-change-default-user/README.md)

* [Use Case 01 - Youtube Video Comments](./src/01-use-case-youtube/README.md)

* [Use Case 02 - Discord Chat Messages](./src/02-use-case-discord/README.md)
