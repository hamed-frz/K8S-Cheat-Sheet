### Use prometheus metrics
    If you're running a production ETCD cluster, you should use Prometheus to monitor ETCD's health and performance. ETCD exposes metrics that can be scraped by Prometheus.
    
- The --metrics=extensive is for prometheus monitoring.
- Can use Auto-Compaction Configuration with --auto-compaction-retention=1h

#### Import key metrics to monitor

    etcd_disk_wal_fsync_duration_seconds: Measures the time it takes to write logs to disk.
    etcd_server_leader_changes_seen_total: Tracks the number of leader changes. High numbers indicate instability.
    etcd_server_proposals_applied_total: Tracks the number of proposals applied by the leader.

#### Configure prometheus to scrape metrics from each ETCD node

```yaml
    scrape_configs:
  - job_name: 'etcd'
    static_configs:
      - targets: ['192.168.100.101:2379', '192.168.100.102:2379', '192.168.100.103:2379']      
```