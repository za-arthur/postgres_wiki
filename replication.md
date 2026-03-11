# Queries

## Replication lag

```sql
select application_name,
    pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), sent_lsn)) send_lag,
    pg_size_pretty(pg_wal_lsn_diff(sent_lsn, replay_lsn)) replay_lag,
    replay_lag
from pg_stat_replication;
```

```sql
select slot_name, slot_type, active,
    case when not pg_is_in_recovery() then pg_size_pretty(pg_current_wal_lsn() - restart_lsn) end as current_lag_bytes,
    case when not pg_is_in_recovery() then pg_size_pretty(pg_current_wal_lsn() - confirmed_flush_lsn) end as current_flush_lag_bytes
from pg_replication_slots s
order by s.slot_name;
```

# Logical replication

## Logical replication slots on standby

Logical replication slots are invalidated in the following cases:

- hot_standby_feedback is off
- hot_standby_feedback is on but there is no a physical slot between
  the primary and the standby. Then, hot_standby_feedback will work,
  but only while the connection is alive (for example a node restart
  would break it)

Then, the primary may delete system catalog rows that could be needed
by the logical decoding on the standby (as it does not know about the
catalog_xmin on the standby).

More [info](https://github.com/postgres/postgres/commit/6af1793954e8c5e753af83c3edb37ed3267dd179).
