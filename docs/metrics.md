# Metrics
This document provides information about retrieving metrics from each
of the EFK components.  Specifically, it provides details about the Prometheus
exposed endpoints.

## Elasticsearch

The Elasticsearch Prometheus endpoint is provided by the [elasticsearch-prometheus-exporter](https://github.com/vvanholl/elasticsearch-prometheus-exporter) plugin.

A sample of the provided metrics:

```
# HELP es_process_mem_total_virtual_bytes Memory used by ES process
# TYPE es_process_mem_total_virtual_bytes gauge
es_process_mem_total_virtual_bytes{cluster="develop",node="develop01",} 3.626733568E9
# HELP es_indices_indexing_is_throttled_bool Is indexing throttling ?
# TYPE es_indices_indexing_is_throttled_bool gauge
es_indices_indexing_is_throttled_bool{cluster="develop",node="develop01",} 0.0
# HELP es_jvm_gc_collection_time_seconds Time spent for GC collections
# TYPE es_jvm_gc_collection_time_seconds counter
es_jvm_gc_collection_time_seconds{cluster="develop",node="develop01",gc="old",} 0.0
es_jvm_gc_collection_time_seconds{cluster="develop",node="develop01",gc="young",} 0.0
# HELP es_indices_requestcache_memory_size_bytes Memory used for request cache
# TYPE es_indices_requestcache_memory_size_bytes gauge
es_indices_requestcache_memory_size_bytes{cluster="develop",node="develop01",} 0.0
# HELP es_indices_search_open_contexts_number Number of search open contexts
# TYPE es_indices_search_open_contexts_number gauge
es_indices_search_open_contexts_number{cluster="develop",node="develop01",} 0.0
# HELP es_jvm_mem_nonheap_used_bytes Memory used apart from heap
# TYPE es_jvm_mem_nonheap_used_bytes gauge
es_jvm_mem_nonheap_used_bytes{cluster="develop",node="develop01",} 5.5302736E7
```
The Prometheus endpoint is secured using the Openshift oauth-proxy and requires specific permissions to
retrieve metrics. Requests must be made to the endpoint using a Service Account that is granted permissions to [`view prometheus`](https://github.com/openshift/openshift-ansible/blob/master/roles/openshift_logging_elasticsearch/templates/2.x/es.j2#L157) in the namespace (e.g `openshift-logging`) where the logging stack is deployed.  The Service
Account name is provided during deployment of the logging stack.

**Note**:
The implementation of metrics is such that requests between the oauth-proxy and Elasticsearch utilize a username and password.  Elasticsearch maintains authorization information separate from Openshift's roles.  Elasticsearch will only allow access to the Service Account provided during deployment of the logging stack.

```
  <oauth token>     ---------------   <username/passwd>   -----------------
    Request   ----> | oauth-proxy | ----- Request ------> | Elasticsearch |
    Response  <---- |             | <---- Respose ------- |               |
                    ---------------                       -----------------
```
### Scrape Rules
A service is deployed for the metrics endpoint which is [annotated](https://github.com/openshift/openshift-ansible/blob/master/roles/openshift_logging_elasticsearch/tasks/main.yaml#L395) to allow auto-discovery of the
endpoint by Prometheus.  The scrape rule must be defined such that it uses the Service Account specified during the deployment of the logging stack.

