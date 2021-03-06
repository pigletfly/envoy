# Listener 发现服务（LDS）

The listener discovery service (LDS) is an optional API that Envoy will call to dynamically fetch listeners. Envoy will reconcile the API response and add, modify, or remove known listeners depending on what is required.

The semantics of listener updates are as follows:

- Every listener must have a unique [name](../../api-v1/listeners/listeners.md#config-listeners-name). If a name is not provided, Envoy will create a UUID. Listeners that are to be dynamically updated should have a unique name supplied by the management server.

- When a listener is added, it will be “warmed” before taking traffic. For example, if the listener references an [RDS](../http_conn_man/rds.md#config-http-conn-man-rds) configuration, that configuration will be resolved and fetched before the listener is moved to “active.”

- Listeners are effectively constant once created. Thus, when a listener is updated, an entirely new listener is created (with the same listen socket). This listener goes through the same warming process described above for a newly added listener.

- When a listener is updated or removed, the old listener will be placed into a “draining” state much like when the entire server is drained for restart. Connections owned by the listener will be gracefully closed (if possible) for some period of time before the listener is removed and any remaining connections are closed. The drain time is set via the [`--drain-time-s`](../../operations/cli.md#cmdoption-drain-time-s) option.

  Note

  Any listeners that are statically defined within the Envoy configuration cannot be modified or removed via the LDS API.

## 配置

- [v1 LDS API](../../api-v1/listeners/lds.md#config-listeners-lds-v1)
- [v2 LDS API](../overview/v2_overview.md#v2-grpc-streaming-endpoints)

## 统计

LDS has a statistics tree rooted at *listener_manager.lds.* with the following statistics:

| Name           | Type    | Description                                                  |
| -------------- | ------- | ------------------------------------------------------------ |
| config_reload  | Counter | Total API fetches that resulted in a config reload due to a different config |
| update_attempt | Counter | Total API fetches attempted                                  |
| update_success | Counter | Total API fetches completed successfully                     |
| update_failure | Counter | Total API fetches that failed (either network or schema errors) |
| version        | Gauge   | Hash of the contents from the last successful API fetch      |