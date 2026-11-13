# Use Case 2: Discord Chat Messages

> Modelling (UDTs)

- Properties:
```
  ---- Message Property ----
  \ status
  \ is_edited
  \ is_pinned

  ---- User ----
  \ uuid
  \ name
  \ profile_picture_url

  ---- Reaction ----
  \ user_uuid
  \ emoji

  ---- Data Envelope ---- 
  \ created_at
  \ updated_at
  \ deleted_at
  \ deletion_reason
```

---

> Modelling (User Chats)

- Partition Keys: `chat_uuid`

- Cluster Keys: `message_uuid`

- Order By: `message_uuid DESC`

- Properties:
```
  \ chat_uuid
  \ message_uuid
  \ server_uuid
  \ message_type
  \ message_content
  \ message_properties
  \ users
  \ reactions
  \ data_envelope
```

---

> Modelling Keyspace, User Defined Types (UDTs) and Table

- https://cassandra.apache.org/doc/stable/cassandra/developing/cql/types.html

- https://cassandra.apache.org/doc/latest/cassandra/reference/cql-commands/create-table.html

- https://cassandra.apache.org/doc/stable/cassandra/architecture/dynamo.html

```cql
CREATE KEYSPACE IF NOT EXISTS discord
WITH replication = { 'class': 'NetworkTopologyStrategy', 'replication_factor': 3 }
  AND durable_writes = true;

USE discord;

CONSISTENCY QUORUM;

CREATE TYPE IF NOT EXISTS message_property (
  status TEXT
  , from_user_uuid UUID
  , is_edited BOOLEAN
  , is_pinned BOOLEAN
);

CREATE TYPE IF NOT EXISTS user (
  uuid UUID
  , name TEXT
  , profile_picture_url TEXT
);

CREATE TYPE IF NOT EXISTS reaction (
  user_uuid UUID
  , emoji TEXT
);

CREATE TYPE IF NOT EXISTS data_envelope (
  created_at TIMESTAMP
  , updated_at TIMESTAMP
  , deleted_at TIMESTAMP
  , deletion_reason TEXT
);

CREATE TABLE IF NOT EXISTS chat_message (
  chat_uuid UUID
  , message_uuid UUID
  , server_uuid UUID
  , message_type TEXT
  , message_content TEXT
  , message_properties message_property
  , users LIST<FROZEN<user>>
  , reactions LIST<FROZEN<reaction>>
  , data_envelope data_envelope

  , PRIMARY KEY ((chat_uuid), message_uuid)
)
WITH CLUSTERING ORDER BY (message_uuid DESC)
  AND cdc = false;
```

---

> Insert Data

```cql
INSERT INTO chat_message (
  chat_uuid
  , message_uuid
  , server_uuid
  , message_type
  , message_content
  , message_properties
  , users
  , reactions
  , data_envelope
)
VALUES (
  01a1d25d-9b2c-774e-a1e0-ebf1b1eb1aa7
  , 01a1d25d-9b2c-774e-a1e0-efaa57c7f232
  , 01a1d25d-9b2c-774e-a1e0-f2cf721dbee5
  , 'TEXT'
  , 'Hey there, how\'s it going?'
  , {
    status: 'SENT'
    , from_user_uuid: 01a1d25d-9b2c-774e-a1e0-f60fc119f971
    , is_edited: FALSE
    , is_pinned: FALSE
  }
  , [
    {
      uuid: 01a1d25d-9b2c-774e-a1e0-f60fc119f971
      , name: 'Yami Fugetsu'
      , profile_picture_url: 'https://api.dicebear.com/9.x/avataaars/svg?seed=user-52148'
    }
    , {
      uuid: 01a1d25d-9b2c-774e-a1e0-f925aacbc27a
      , name: 'Goku'
      , profile_picture_url: 'https://api.dicebear.com/9.x/avataaars/svg?seed=user-02186'
    }
  ]
  , [
    {
      user_uuid: 01a1d25d-9b2c-774e-a1e0-f925aacbc27a
      , emoji: '👍'
    }
  ]
  , {
    created_at: currentTimestamp()
    , updated_at: currentTimestamp()
    , deleted_at: NULL
    , deletion_reason: NULL
  }
);
```