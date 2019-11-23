- standard go runtime metrics prefixed by `go_`
- process level metrics prefixed with `process_`
- prometheus scrap metrics prefixed with `promhttp_`
- `minio_disk_storage_used_bytes` : Total byte count of disk storage used by current MinIO server instance
- `minio_http_requests_duration_seconds_bucket` : Cumulative counters for all the request types (HEAD/GET/PUT/POST/DELETE) in different time brackets
- `minio_http_requests_duration_seconds_count` : Count of current number of observations i.e. total HTTP requests (HEAD/GET/PUT/POST/DELETE)
- `minio_http_requests_duration_seconds_sum` : Current aggregate time spent servicing all HTTP requests (HEAD/GET/PUT/POST/DELETE) in seconds
- `minio_network_received_bytes_total` : Total number of bytes received by current MinIO server instance
- `minio_network_sent_bytes_total` : Total number of bytes sent by current MinIO server instance
- `minio_offline_disks` : Total number of offline disks for current MinIO server instance
- `minio_total_disks` : Total number of disks for current MinIO server instance
- `minio_disk_storage_available_bytes` : Current storage space available to MinIO server in bytes
- `minio_disk_storage_total_bytes` : Total storage space available to MinIO server in bytes
- `process_start_time_seconds` : Start time of MinIO server since unix epoc hin seconds

If you're running MinIO gateway, disk/storage information is not exposed. Only following metrics are available

- `minio_http_requests_duration_seconds_bucket` : Cumulative counters for all the request types (HEAD/GET/PUT/POST/DELETE) in different time brackets
- `minio_http_requests_duration_seconds_count` : Count of current number of observations i.e. total HTTP requests (HEAD/GET/PUT/POST/DELETE)
- `minio_http_requests_duration_seconds_sum` : Current aggregate time spent servicing all HTTP requests (HEAD/GET/PUT/POST/DELETE) in seconds
- `minio_network_received_bytes_total` : Total number of bytes received by current MinIO server instance
- `minio_network_sent_bytes_total` : Total number of bytes sent by current MinIO server instance
- `process_start_time_seconds` : Start time of MinIO server since unix epoch in seconds

For MinIO instances with [`caching`](https://github.com/minio/minio/tree/master/docs/disk-caching) enabled, these additional metrics are available.

- `minio_disk_cache_storage_bytes` : Total byte count of cache capacity available for current MinIO server instance
- `minio_disk_cache_storage_free_bytes` : Total byte count of free cache available for current MinIO server instance