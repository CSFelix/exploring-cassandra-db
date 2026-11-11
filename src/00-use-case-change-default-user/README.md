# Use Case 0: Replace Default User and Password

> Default Credentials

- User: `cassandra`
- Password: `cassandra`

---

> Log into CQLSH (Cassandra Query Language Shell)

```bash
docker run -it cassandra-zero bash
cqlsh -u cassandra
```

---

> Check Cluster and Nodes (Peers)

```sql
SELECT cluster_name, cql_version, data_center, partitioner, rack FROM system.local; # cluster

SELECT peer, data_center, rack, schema_version FROM system.peers_v2; # recognized nodes in the cluster by the context cluster
```

---

> Check Cassandra Rings

```bash
nodetool ring
```

---

> Create new Role/User and Revoke Default Role

- https://cassandra.apache.org/doc/4.1/cassandra/cql/security.html

- https://cassandra.apache.org/doc/latest/cassandra/tooling/hash-password.html

```sql
LIST ROLES;
LIST USERS;
SELECT * FROM system_auth.roles;

CREATE ROLE IF NOT EXISTS admin
WITH SUPERUSER = true
	AND LOGIN = true
	AND PASSWORD = '<a-strong-password>'
	-- AND <HASHED> PASSWORD = '<a-bcrypt-hashed-password>'
	AND ACCESS TO ALL DATACENTERS;

-- log into 'admin' role

DROP ROLE cassandra;

LIST ROLES;
LIST USERS;
SELECT * FROM system_auth.roles;

ALTER KEYSPACE system_auth WITH replication = { 'class': 'NetworkTopologyStrategy', 'replication_factor': 3 } AND durable_writes = true;
```

---

> Repair Nodes

Since a system keyspace has been updated with replication, it is needed to run a repair to all nodes in order to propagate the update.

```bash
nodetool repair --full
```
