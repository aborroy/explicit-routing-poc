# PoC for Explicit Routing

This project contains a Docker Compose template for Explicit Routing with a custom content model and two shards.

Content model detail:

```xml
<aspect name="shard:sharding">
    <properties>
        <!-- Shard number to index this content -->
        <property name="shard:shardId">
            <type>d:text</type>
        </property>
    </properties>
</aspect>
```

# Using Docker Compose

Just start it as usual

```
$ docker-compose up --build
```

Alfresco will be available at:

http://localhost:8081/alfresco

http://localhost:8080/share

http://localhost:8083/solr

http://localhost:8084/solr

# Configure Share web app

http://localhost:8080/share

Create two folders:

* Sharding-0
* Sharding-1

Add following rule to each folder.

Sharding-0

```
Sharding Aspect
Description:
Active
Run in background
Rule applied to subfolders
When:
Items are created or enter this folder

If all criteria are met:
All Items

Perform Action:
Set property valueProperty:shard:shardId Value:0
```

Sharding-1

```
Sharding Aspect
Description:
Active
Run in background
Rule applied to subfolders
When:
Items are created or enter this folder

If all criteria are met:
All Items

Perform Action:
Set property valueProperty:shard:shardId Value:1
```

Documents uploaded to "Sharding-0" will have shardId to 0 and the ones uploaded to "Sharding-1" folder will have shardId to 1.
