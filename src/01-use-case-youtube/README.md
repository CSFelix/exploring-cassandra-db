# Use Case 1: Youtube Video Comments

> Modelling

- Partition Keys: `video_uuid`

- Cluster Keys: `comment_uuid`

- Order By: `comment_uuid DESC`

- Properties:
```
	---- Keyspace Properties ----
	\ video_uuid (Partition Key)
	\ comment_uuid (Cluster Key)
	\ video_publisher_uuid
	\ comment
	\ likes
	\ dislikes
	\ user_uuid
	\ user_name
	\ user_profile_picture_url
	\ is_edited
	\ is_reply
	\ replies
	\ data_envelope (UDT)

	---- UDT Properties ----
	\ created_at
	\ updated_at
	\ deleted_at
	\ deletion_reason
```

---

> Model Keyspace, User Defined Types (UDTs) and Table

- https://cassandra.apache.org/doc/stable/cassandra/developing/cql/types.html

- https://cassandra.apache.org/doc/latest/cassandra/reference/cql-commands/create-table.html

```cql
CREATE KEYSPACE IF NOT EXISTS youtube
WITH replication = { 'class': 'NetworkTopologyStrategy', 'replication_factor': 3, 'datacenter-1': 3 } -- added 'datacenter-1' key-value just to know it is possible to set up replciation_factor per datacenter
 AND durable_writes = true;

USE youtube;

CREATE TYPE IF NOT EXISTS data_envelope(
	created_at TIMESTAMP
	, updated_at TIMESTAMP
	, deleted_at TIMESTAMP
	, deletion_reason TEXT
);

CREATE TABLE IF NOT EXISTS comment_by_video_uuid(
	video_uuid UUID
	, video_publisher_uuid UUID
	, comment_uuid UUID
	, comment TEXT
	, likes SET<UUID>
	, dislikes SET<UUID>
	, user_uuid UUID
	, user_name TEXT
	, user_profile_picture_url TEXT
	, is_edited BOOLEAN
	, is_reply BOOLEAN
	, replies SET<UUID>
	, data_envelope data_envelope

	, PRIMARY KEY ((video_uuid), comment_uuid)
)
WITH CLUSTERING ORDER BY (comment_uuid DESC)
  AND cdc = false;
```

---

> Insert Data

- https://cassandra.apache.org/doc/stable/cassandra/architecture/dynamo.html

```cql
CONSISTENCY QUORUM;

INSERT INTO comment_by_video_uuid(
	video_uuid
	, video_publisher_uuid
	, comment_uuid
	, comment
	, likes
	, dislikes
	, user_uuid
	, user_name
	, user_profile_picture_url
	, is_edited
	, is_reply
	, replies
	, data_envelope
)
VALUES (
	01a1c180-407c-7ae2-9a09-7b7742e7328b
	, 01a1c180-407c-746c-885b-2593f9752c55
	, 01a1c180-407c-7f19-afb7-f90de9f946fe
	, 'Comment 1'
	, NULL
	, NULL
	, 01a1c180-407c-72af-9a33-044de79cdfad
	, 'Yami Fugetsu'
	, NULL
	, FALSE
	, FALSE
	, NULL
	, {
		created_at: currentTimestamp()
		, updated_at: currentTimestamp()
		, deleted_at: NULL
		, deletion_reason: NULL
	}
)
IF NOT EXISTS;
```

---

> Populate Table

```bash
docker cp ./comments-seed.cql cassandra-zero:/tmp/comments-seed.cql
docker exec -it cassandra-zero bash
cd ./tmp
cqlsh -u admin -k youtube_video < comments-seed.cql
rm comments-seed.cql
```

```cql
SELECT DISTINCT video_uuid FROM comment_by_video_uuid;
SELECT COUNT(*) FROM comment_by_video_uuid GROUP BY video_uuid;
```

---

> Fetching Data

```cql
-- Without Pagination (page: 1)
SELECT *
FROM comment_by_video_uuid
WHERE video_uuid = 01a1c4fa-85af-7442-99d7-90cb1df0fad0
ORDER BY comment_uuid DESC -- ASC | DESC
LIMIT 50;

-- With Pagination (page: 2+)
SELECT *
FROM comment_by_video_uuid
WHERE video_uuid = 01a1c4fa-85af-7442-99d7-90cb1df0fad0
	AND comment_uuid < 01a1c4fa-9b31-7159-8e93-45b1d7d2f4c7
ORDER BY comment_uuid DESC -- ASC | DESC
LIMIT 50;
```

---

> Search Partition Key in Ring

```bash
# nodetool getendpoints <keyspace> <table> '<partition_key>'
nodetool getendpoints youtube comment_by_video_uuid '01a1c4fa-85af-7442-99d7-90cb1df0fad0'
```
