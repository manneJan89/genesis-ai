# Performance: server profile
APIs and backends.

- **Never load unbounded result sets.** List and query endpoints paginate
  (limit plus cursor or offset) with a maximum page size enforced on the server.
- **Every filter and sort is backed by an index**; flag any query on an
  unindexed field.
- **No N+1 access.** Loading a list and then querying once per item is a
  defect; batch (`$in`, `IN (...)`, `DataLoader`) or join.
- **Filter at the query**, not in memory after loading everything.
- **Project only the fields you need** from the database and in responses.
- **Stateless request handling:** state lives in the data store or cache, not in
  process memory, so the service can run as more than one instance.
- **Caching is per-user when the data is.** A cache key that leaves out the user
  or tenant serves one user's data to another: that's a security bug, not a
  performance win.
- **Timeouts on every outbound call**, and no unbounded retries.
