escp - An ElasticSearch Copy Utility


escp is a python script to aid in re-indexing or copying indices between or within ES clusters. It uses the official ElasticSearch Python API to expose a user-friendly interface and hence requires the 'elasticsearch' python package to be installed (via pip or otherwise).


```
usage: escp [-h] [-i] [-s] [-c] [-m] [-w WORKERS] [-r SRCREGEX]
            [-C CHUNK_SIZE] [-S SHARDS] [-R REPLICAS]
            sourceIndex destIndex

Copy indices within or between ElasticSearch clusters

positional arguments:
  sourceIndex           The source cluster and index to transfer, e.g.,
                        http://localhost:9200/my.index
  destIndex             The destination cluster and index to transfer to,
                        e.g., http://localhost:9200/my.destindex

optional arguments:
  -h, --help            show this help message and exit
  -i, --ignore-health   Ignore the health status of the input and output
                        servers
  -s, --strict          Destination Index must *not* exist and must be created
  -c, --create-only     Only create new entries, if an entry with a given id
                        exists already it will not be overwritten/updated
  -m, --no-mapping      Do not copy mapping over from source
  -w WORKERS, --workers WORKERS
                        Number of bulk workers to run
  -r SRCREGEX, --source-regex SRCREGEX
                        Use formating for destination best on source index
                        regex
  -C CHUNK_SIZE, --chunk-size CHUNK_SIZE
                        Number of docs to chunk up and send to destintation
  -S SHARDS, --shards SHARDS
                        Number of shards for output index
  -R REPLICAS, --replicas REPLICAS
                        Number of replicas for output index

```
Examples:

To copy an index within the same cluster on localhost, ensuring the output index doesn't already exist, it would look like:

```
escp -s localhost:9200/input_index localhost:9200/output_index
```

Copy an index within the same cluster, ensuring entries are only created (default behavior is to replace/update if exists)

```
escp -c localhost:9200/input_index localhost:9200/output_index
```

Copy multiple indices into a single index

```
escp localhost:9200/input_indices-* localhost:9200/output_index
```

Copy multiple indices from one cluster to another

```
escp localhost:9200/my.indices localhost:9201/
```

Copy multiple indices into multiple indices based on the source index name. Using the '\*' character you can copy the source index name into the destination. In the below example all destination indices will have the same name as the source index but preceeded with 'copy-\*'

```
escp localhost:9200/input_indices-* localhost:9200/copy-*
```

For more advanced usage you can use the -r or --source-regex flag to parse the source index name and use the parsed components as part of the destination name. For example if you have a set of time series indices you can change the prefix like so

```
escp -r "logstash\-(\d+)\.(\d+)\.(\d+)" localhost:9200/logstash-2016.01.* localhost:9200/output-?{1}.?{2}.?{3}"
```

This would take all the indices that match the source query and reuse the date fields in the destination. Note that the regex is python regex syntax and uses the groups that match (the things in parens) to replace the fields in the destination. The numbers begin with 1 and go up to the number of matches you expect in your regex. You can use the matches using the above syntax which looks like ?{\<number\>} where number corresponds the regexes matching group.
