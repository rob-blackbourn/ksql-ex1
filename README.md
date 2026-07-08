# ksql-ex1

## 1. Start the server

### docker

```bash
docker-compose up
```

### podman

```bash
podman compose up
```

## 2. Start the client

In a separate window start the command line client.

### Docker

```bash
docker exec -it ksqldb-cli ksql http://ksqldb-server:8088
```

### Podman

```bash
podman exec -it ksqldb-cli ksql http://ksqldb-server:8088
```


## 3. Create a Stream

```sql
CREATE STREAM riderLocations
(
    profileId   VARCHAR,
    latitude    DOUBLE,
    longitude   DOUBLE
)
WITH
(
    kafka_topic='locations',
    value_format='json',
    partitions=1
);
```

Create a table.

```sql
CREATE TABLE currentLocation
AS
    SELECT
        profileId,
        LATEST_BY_OFFSET(latitude) AS la,
        LATEST_BY_OFFSET(longitude) AS lo
    FROM
        riderlocations
    GROUP BY
        profileId
    EMIT CHANGES;
```

Create a derived table.

```sql
-- Create the ridersNearMountainView table
CREATE TABLE ridersNearMountainView
AS
    SELECT
        ROUND(GEO_DISTANCE(la, lo, 37.4133, -122.1162), -1) AS distanceInMiles,
        COLLECT_LIST(profileId) AS riders,
        COUNT(*) AS count
    FROM
        currentLocation
    GROUP BY
        ROUND(GEO_DISTANCE(la, lo, 37.4133, -122.1162), -1);
```

Run a push query.

```sql
SELECT
    *
FROM
    riderLocations
WHERE
    GEO_DISTANCE(latitude, longitude, 37.4133, -122.1162) <= 5
EMIT CHANGES;
```

Insert some data on a new connection.

```sql
INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('c2309eec', 37.7877, -122.4205);
INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('18f4ea86', 37.3903, -122.0643);
INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('4ab5cbad', 37.3952, -122.0813);
INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('8b6eae59', 37.3944, -122.0813);
INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('4a7c7b41', 37.4049, -122.0822);
INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('4ddad000', 37.7857, -122.4011);
```

Run a pull query.

```sql
SELECT * from ridersNearMountainView WHERE distanceInMiles <= 10;
```
