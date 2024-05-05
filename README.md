# clickhouse-helmchart

this is a clickhouse helm chart configured in such a way that when there is only 20 percent free space remaining in the hot disc the data will move to cold disc

## Steps for testing

1. cd to the clickhouse-chart directory
2. run the command `helm install clickhouse-release .`
3. then we need to exec into the container you can also do it using the service defined with the helm chart but for the time being we can directly exec into the container using the command `kubectl exec -it <Pod name> clickhouse-client`
4. Once you are into the container we need to make a table dz_test by following method : `CREATE TABLE dz_test
(
    B Int64,
    T String,
    D Date
)
ENGINE = MergeTree
PARTITION BY D
ORDER BY B
SETTINGS storage_policy = 'hot_cold_policy'`
5. once the table is created we need to insert a large amount od data into it for that run the command : `INSERT INTO dz_test SELECT
    number,
    number,
    '2023-01-01'
FROM numbers(1000000000.)` this will insert 1 billion rows in the table 
6. to check for the disc which is being used run `SELECT
    disk_name,
    formatReadableSize(sum(bytes_on_disk)) AS total_size
FROM system.parts
WHERE table = 'dz_test'
GROUP BY disk_name` initially you will see only hot is being used but when you will run the query again after sometime you will notice that data is moved to the cold storage . 