Debezium
---

1. Start up containers

	```sh
	docker-compose up
	```

2. Check if the connector is up

	```sh
	curl -H "Accept:application/json" localhost:8083/
	```

	```json
	{
	  "version": "1.1.0",
	  "commit": "fdcf75ea326b8e07",
	  "kafka_cluster_id": "DnFLwG2wQqO-kUbtCcCE_g"
	}
	```

3. Check the current connectors

	```sh
	curl -H "Accept:application/json" localhost:8083/connectors/
	```
	
	```json
	[]
	```

4. Create a new mysql connection

	```sh
	curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.whitelist": "inventory", "database.history.kafka.bootstrap.servers": "kafka:9092", "database.history.kafka.topic": "dbhistory.inventory" } }'
	```
	
	```json
	{
	  "name": "inventory-connector",
	  "config": {
	    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
	    "tasks.max": "1",
	    "database.hostname": "mysql",
	    "database.port": "3306",
	    "database.user": "debezium",
	    "database.password": "dbz",
	    "database.server.id": "184054",
	    "database.server.name": "dbserver1",
	    "database.whitelist": "inventory",
	    "database.history.kafka.bootstrap.servers": "kafka:9092",
	    "database.history.kafka.topic": "dbhistory.inventory",
	    "name": "inventory-connector"
	  },
	  "tasks": [],
	  "type": null
	}
	```

5. Check the topics

	```sh
	docker exec -it <kafka_container_id> bin/sh
	```

	```sh
	bin/kafka-topics.sh --list --zookeeper zookeeper:2181
	```
	
	```
	__consumer_offsets
	connect-status
	dbhistory.inventory
	dbserver1
	dbserver1.inventory.addresses
	dbserver1.inventory.customers
	dbserver1.inventory.orders
	dbserver1.inventory.products
	dbserver1.inventory.products_on_hand
	my_connect_configs
	my_connect_offsets
	```

6. Fetch change events

	a. Start up an event watcher:

	```sh
	docker run -it --name watcher --rm --network cdc_default --link cdc_zookeeper_1:zookeeper -e ZOOKEEPER_CONNECT=zookeeper:2181 debezium/kafka watch-topic -a -k dbserver1.inventory.customers
	```

	b. Update field:

	```sql
	UPDATE customers SET first_name='Anne Marie' WHERE id=1004;
	```

	c. Event Value:

	```json
	{
	  "schema": {
	    "type": "struct",
	    "fields": [
	      {
	        "type": "struct",
	        "fields": [
	          {
	            "type": "int32",
	            "optional": false,
	            "field": "id"
	          },
	          {
	            "type": "string",
	            "optional": false,
	            "field": "first_name"
	          },
	          {
	            "type": "string",
	            "optional": false,
	            "field": "last_name"
	          },
	          {
	            "type": "string",
	            "optional": false,
	            "field": "email"
	          }
	        ],
	        "optional": true,
	        "name": "dbserver1.inventory.customers.Value",
	        "field": "before"
	      },
	      {
	        "type": "struct",
	        "fields": [
	          {
	            "type": "int32",
	            "optional": false,
	            "field": "id"
	          },
	          {
	            "type": "string",
	            "optional": false,
	            "field": "first_name"
	          },
	          {
	            "type": "string",
	            "optional": false,
	            "field": "last_name"
	          },
	          {
	            "type": "string",
	            "optional": false,
	            "field": "email"
	          }
	        ],
	        "optional": true,
	        "name": "dbserver1.inventory.customers.Value",
	        "field": "after"
	      },
	      {
	        "type": "struct",
	        "fields": [
	          {
	            "type": "string",
	            "optional": true,
	            "field": "version"
	          },
	          {
	            "type": "string",
	            "optional": false,
	            "field": "name"
	          },
	          {
	            "type": "int64",
	            "optional": false,
	            "field": "server_id"
	          },
	          {
	            "type": "int64",
	            "optional": false,
	            "field": "ts_sec"
	          },
	          {
	            "type": "string",
	            "optional": true,
	            "field": "gtid"
	          },
	          {
	            "type": "string",
	            "optional": false,
	            "field": "file"
	          },
	          {
	            "type": "int64",
	            "optional": false,
	            "field": "pos"
	          },
	          {
	            "type": "int32",
	            "optional": false,
	            "field": "row"
	          },
	          {
	            "type": "boolean",
	            "optional": true,
	            "default": false,
	            "field": "snapshot"
	          },
	          {
	            "type": "int64",
	            "optional": true,
	            "field": "thread"
	          },
	          {
	            "type": "string",
	            "optional": true,
	            "field": "db"
	          },
	          {
	            "type": "string",
	            "optional": true,
	            "field": "table"
	          },
	          {
	            "type": "string",
	            "optional": true,
	            "field": "query"
	          }
	        ],
	        "optional": false,
	        "name": "io.debezium.connector.mysql.Source",
	        "field": "source"
	      },
	      {
	        "type": "string",
	        "optional": false,
	        "field": "op"
	      },
	      {
	        "type": "int64",
	        "optional": true,
	        "field": "ts_ms"
	      }
	    ],
	    "optional": false,
	    "name": "dbserver1.inventory.customers.Envelope"
	  },
	  "payload": {
	    "before": {
	      "id": 1004,
	      "first_name": "Anne",
	      "last_name": "Kretchmar",
	      "email": "annek@noanswer.org"
	    },
	    "after": {
	      "id": 1004,
	      "first_name": "Anne Marie",
	      "last_name": "Kretchmar",
	      "email": "annek@noanswer.org"
	    },
	    "source": {
	      "version": "0.8.3.Final",
	      "name": "dbserver1",
	      "server_id": 223344,
	      "ts_sec": 1540326308,
	      "gtid": null,
	      "file": "mysql-bin.000003",
	      "pos": 364,
	      "row": 0,
	      "snapshot": false,
	      "thread": 15,
	      "db": "inventory",
	      "table": "customers",
	      "query": null
	    },
	    "op": "u",
	    "ts_ms": 1540326308383
	  }
	}
	```
