# Hashicorp Vault

This integration collects logs and metrics from Hashicorp Vault. There are
three data streams:

- audit - Audit logs from file or TCP socket.
- log - Operation log from file.
- metrics - Telemetry data from the /sys/metrics API.

## Compatibility

This integration has been tested with Vault 1.11.

## Audit Logs

Vault audit logs provide a detailed accounting of who accessed or modified what
secrets. The logs do not contain the actual secret values (for strings), but
instead contain the value hashed with a salt using HMAC-SHA256. Hashes can be
compared to values by using the
[`/sys/audit-hash`](https://www.vaultproject.io/api/system/audit-hash.html) API.

In order to use this integration for audit logs you must configure Vault
to use a [`file` audit device](https://www.vaultproject.io/docs/audit/file)
or [`socket` audit device](https://www.vaultproject.io/docs/audit/socket). The
file audit device provides the strongest delivery guarantees.

### File audit device requirements

- Create a directory for audit logs on each Vault server host.

```
mkdir /var/log/vault
```

- Enable the file audit device.

```
vault audit enable file file_path=/var/log/vault/audit.json
```

- Configure log rotation for the audit log. The exact steps may vary by OS.
This example uses `logrotate` to call `systemctl reload` on the
[Vault service](https://learn.hashicorp.com/tutorials/vault/deployment-guide#step-3-configure-systemd)
which sends the process a SIGHUP signal. The SIGHUP signal causes Vault to start
writing to a new log file.

```
tee /etc/logrotate.d/vault <<'EOF'
/var/log/vault/audit.json {
    rotate 7
    daily
    compress
    delaycompress
    missingok
    notifempty
    extension json
    dateext
    dateformat %Y-%m-%d.
    postrotate
        /bin/systemctl reload vault || true
    endscript
}
EOF
```

### Socket audit device requirements

To enable the socket audit device in Vault you should first enable this
integration because Vault will test that it can connect to the TCP socket.

- Add this integration and enable audit log collection via TCP. If Vault will
be connecting remotely set the listen address to 0.0.0.0.

- Configure the socket audit device to stream logs to this integration.
Substitute in the IP address of the Elastic Agent to which you are sending the
audit logs.

```
vault audit enable socket address=${ELASTIC_AGENT_IP}:9007 socket_type=tcp
```

An example event for `audit` looks as following:

```json
{
    "@timestamp": "2023-09-26T13:07:49.743Z",
    "agent": {
        "ephemeral_id": "5bbd86cc-8032-432d-be82-fae8f624ed98",
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "name": "docker-fleet-agent",
        "type": "filebeat",
        "version": "8.10.1"
    },
    "data_stream": {
        "dataset": "hashicorp_vault.audit",
        "namespace": "ep",
        "type": "logs"
    },
    "ecs": {
        "version": "8.17.0"
    },
    "elastic_agent": {
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "snapshot": false,
        "version": "8.10.1"
    },
    "event": {
        "action": "update",
        "agent_id_status": "verified",
        "category": [
            "authentication"
        ],
        "dataset": "hashicorp_vault.audit",
        "id": "0b1b9013-da54-633d-da69-8575e6794ed3",
        "ingested": "2023-09-26T13:08:15Z",
        "kind": "event",
        "original": "{\"time\":\"2023-09-26T13:07:49.743284857Z\",\"type\":\"request\",\"auth\":{\"token_type\":\"default\"},\"request\":{\"id\":\"0b1b9013-da54-633d-da69-8575e6794ed3\",\"operation\":\"update\",\"namespace\":{\"id\":\"root\"},\"path\":\"sys/audit/test\"}}",
        "outcome": "success",
        "type": [
            "info"
        ]
    },
    "hashicorp_vault": {
        "audit": {
            "auth": {
                "token_type": "default"
            },
            "request": {
                "id": "0b1b9013-da54-633d-da69-8575e6794ed3",
                "namespace": {
                    "id": "root"
                },
                "operation": "update",
                "path": "sys/audit/test"
            },
            "type": "request"
        }
    },
    "host": {
        "architecture": "x86_64",
        "containerized": false,
        "hostname": "docker-fleet-agent",
        "id": "28da52b32df94b50aff67dfb8f1be3d6",
        "ip": [
            "192.168.80.5"
        ],
        "mac": [
            "02-42-C0-A8-50-05"
        ],
        "name": "docker-fleet-agent",
        "os": {
            "codename": "focal",
            "family": "debian",
            "kernel": "5.10.104-linuxkit",
            "name": "Ubuntu",
            "platform": "ubuntu",
            "type": "linux",
            "version": "20.04.6 LTS (Focal Fossa)"
        }
    },
    "input": {
        "type": "log"
    },
    "log": {
        "file": {
            "path": "/tmp/service_logs/vault/audit.json"
        },
        "offset": 0
    },
    "tags": [
        "preserve_original_event",
        "hashicorp-vault-audit"
    ]
}
```

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| ecs.version | ECS version this event conforms to. `ecs.version` is a required field and must exist in all events. When querying across multiple indices -- which may conform to slightly different ECS versions -- this field lets integrations adjust to the schema version of the events. | keyword |
| event.action | The action captured by the event. This describes the information in the event. It is more specific than `event.category`. Examples are `group-add`, `process-started`, `file-created`. The value is normally defined by the implementer. | keyword |
| event.category | This is one of four ECS Categorization Fields, and indicates the second level in the ECS category hierarchy. `event.category` represents the "big buckets" of ECS categories. For example, filtering on `event.category:process` yields all events relating to process activity. This field is closely related to `event.type`, which is used as a subcategory. This field is an array. This will allow proper categorization of some events that fall in multiple categories. | keyword |
| event.dataset | Event dataset | constant_keyword |
| event.id | Unique ID to describe the event. | keyword |
| event.kind | This is one of four ECS Categorization Fields, and indicates the highest level in the ECS category hierarchy. `event.kind` gives high-level information about what type of information the event contains, without being specific to the contents of the event. For example, values of this field distinguish alert events from metric events. The value of this field can be used to inform how these kinds of events should be handled. They may warrant different retention, different access control, it may also help understand whether the data is coming in at a regular interval or not. | keyword |
| event.module | Event module | constant_keyword |
| event.original | Raw text message of entire event. Used to demonstrate log integrity or where the full log message (before splitting it up in multiple parts) may be required, e.g. for reindex. This field is not indexed and doc_values are disabled. It cannot be searched, but it can be retrieved from `_source`. If users wish to override this and index this field, please see `Field data types` in the `Elasticsearch Reference`. | keyword |
| event.outcome | This is one of four ECS Categorization Fields, and indicates the lowest level in the ECS category hierarchy. `event.outcome` simply denotes whether the event represents a success or a failure from the perspective of the entity that produced the event. Note that when a single transaction is described in multiple events, each event may populate different values of `event.outcome`, according to their perspective. Also note that in the case of a compound event (a single event that contains multiple logical events), this field should be populated with the value that best captures the overall success or failure from the perspective of the event producer. Further note that not all events will have an associated outcome. For example, this field is generally not populated for metric events, events with `event.type:info`, or any events for which an outcome does not make logical sense. | keyword |
| event.type | This is one of four ECS Categorization Fields, and indicates the third level in the ECS category hierarchy. `event.type` represents a categorization "sub-bucket" that, when used along with the `event.category` field values, enables filtering events down to a level appropriate for single visualization. This field is an array. This will allow proper categorization of some events that fall in multiple event types. | keyword |
| hashicorp_vault.audit.auth.accessor | This is an HMAC of the client token accessor | keyword |
| hashicorp_vault.audit.auth.client_token | This is an HMAC of the client's token ID. | keyword |
| hashicorp_vault.audit.auth.display_name | Display name is a non-security sensitive identifier that is applicable to this auth. It is used for logging and prefixing of dynamic secrets. For example, it may be "armon" for the github credential backend. If the client token is used to generate a SQL credential, the user may be "github-armon-uuid". This is to help identify the source without using audit tables. | keyword |
| hashicorp_vault.audit.auth.entity_id | Entity ID is the identifier of the entity in identity store to which the identity of the authenticating client belongs to. | keyword |
| hashicorp_vault.audit.auth.external_namespace_policies | External namespace policies represent the policies authorized from different namespaces indexed by respective namespace identifiers. | flattened |
| hashicorp_vault.audit.auth.identity_policies | These are the policies sourced from the identity. | keyword |
| hashicorp_vault.audit.auth.metadata | This will contain a list of metadata key/value pairs associated with the authenticated user. | flattened |
| hashicorp_vault.audit.auth.no_default_policy | Indicates that the default policy should not be added by core when creating a token. The default policy will still be added if it's explicitly defined. | boolean |
| hashicorp_vault.audit.auth.policies | Policies is the list of policies that the authenticated user is associated with. | keyword |
| hashicorp_vault.audit.auth.policy_results.allowed |  | boolean |
| hashicorp_vault.audit.auth.policy_results.granting_policies.name |  | keyword |
| hashicorp_vault.audit.auth.policy_results.granting_policies.namespace_id |  | keyword |
| hashicorp_vault.audit.auth.policy_results.granting_policies.type |  | keyword |
| hashicorp_vault.audit.auth.remaining_uses |  | long |
| hashicorp_vault.audit.auth.token_issue_time |  | date |
| hashicorp_vault.audit.auth.token_policies | These are the policies sourced from the token. | keyword |
| hashicorp_vault.audit.auth.token_ttl |  | long |
| hashicorp_vault.audit.auth.token_type |  | keyword |
| hashicorp_vault.audit.entity_created | entity_created is set to true if an entity is created as part of a login request. | boolean |
| hashicorp_vault.audit.error | If an error occurred with the request, the error message is included in this field's value. | keyword |
| hashicorp_vault.audit.request.client_certificate_serial_number | Serial number from the client's certificate. | keyword |
| hashicorp_vault.audit.request.client_id |  | keyword |
| hashicorp_vault.audit.request.client_token | This is an HMAC of the client's token ID. | keyword |
| hashicorp_vault.audit.request.client_token_accessor | This is an HMAC of the client token accessor. | keyword |
| hashicorp_vault.audit.request.data | The data object will contain secret data in key/value pairs. | flattened |
| hashicorp_vault.audit.request.headers | Additional HTTP headers specified by the client as part of the request. | flattened |
| hashicorp_vault.audit.request.id | This is the unique request identifier. | keyword |
| hashicorp_vault.audit.request.mount_accessor |  | keyword |
| hashicorp_vault.audit.request.mount_type |  | keyword |
| hashicorp_vault.audit.request.namespace.id |  | keyword |
| hashicorp_vault.audit.request.namespace.path |  | keyword |
| hashicorp_vault.audit.request.operation | This is the type of operation which corresponds to path capabilities and is expected to be one of: create, read, update, delete, or list. | keyword |
| hashicorp_vault.audit.request.path | The requested Vault path for operation. | keyword |
| hashicorp_vault.audit.request.path.text | Multi-field of `hashicorp_vault.audit.request.path`. | text |
| hashicorp_vault.audit.request.policy_override | Policy override indicates that the requestor wishes to override soft-mandatory Sentinel policies. | boolean |
| hashicorp_vault.audit.request.remote_address | The IP address of the client making the request. | ip |
| hashicorp_vault.audit.request.remote_port | The remote port of the client making the request. | long |
| hashicorp_vault.audit.request.replication_cluster | Name given to the replication secondary where this request originated. | keyword |
| hashicorp_vault.audit.request.wrap_ttl | If the token is wrapped, this displays configured wrapped TTL in seconds. | long |
| hashicorp_vault.audit.response.auth.accessor |  | keyword |
| hashicorp_vault.audit.response.auth.client_token |  | keyword |
| hashicorp_vault.audit.response.auth.display_name |  | keyword |
| hashicorp_vault.audit.response.auth.entity_id |  | keyword |
| hashicorp_vault.audit.response.auth.external_namespace_policies |  | flattened |
| hashicorp_vault.audit.response.auth.identity_policies |  | keyword |
| hashicorp_vault.audit.response.auth.metadata |  | flattened |
| hashicorp_vault.audit.response.auth.no_default_policy |  | boolean |
| hashicorp_vault.audit.response.auth.num_uses |  | long |
| hashicorp_vault.audit.response.auth.policies |  | keyword |
| hashicorp_vault.audit.response.auth.token_issue_time |  | date |
| hashicorp_vault.audit.response.auth.token_policies |  | keyword |
| hashicorp_vault.audit.response.auth.token_ttl | Time to live for the token in seconds. | long |
| hashicorp_vault.audit.response.auth.token_type |  | keyword |
| hashicorp_vault.audit.response.data | Response payload. | flattened |
| hashicorp_vault.audit.response.headers | Headers will contain the http headers from the plugin that it wishes to have as part of the output. | flattened |
| hashicorp_vault.audit.response.mount_accessor |  | keyword |
| hashicorp_vault.audit.response.mount_type |  | keyword |
| hashicorp_vault.audit.response.redirect | Redirect is an HTTP URL to redirect to for further authentication. This is only valid for credential backends. This will be blanked for any logical backend and ignored. | keyword |
| hashicorp_vault.audit.response.warnings |  | keyword |
| hashicorp_vault.audit.response.wrap_info.accessor | The token accessor for the wrapped response token. | keyword |
| hashicorp_vault.audit.response.wrap_info.creation_path | Creation path is the original request path that was used to create the wrapped response. | keyword |
| hashicorp_vault.audit.response.wrap_info.creation_time | The creation time. This can be used with the TTL to figure out an expected expiration. | date |
| hashicorp_vault.audit.response.wrap_info.token | The token containing the wrapped response. | keyword |
| hashicorp_vault.audit.response.wrap_info.ttl | Specifies the desired TTL of the wrapping token. | long |
| hashicorp_vault.audit.response.wrap_info.wrapped_accessor | The token accessor for the wrapped response token. | keyword |
| hashicorp_vault.audit.type | Audit record type (request or response). | keyword |
| input.type |  | keyword |
| log.file.path | Full path to the log file this event came from, including the file name. It should include the drive letter, when appropriate. If the event wasn't read from a log file, do not populate this field. | keyword |
| log.offset |  | long |
| log.source.address | Source address (IP and port) of the log message. | keyword |
| message | For log events the message field contains the log message, optimized for viewing in a log viewer. For structured logs without an original message field, other fields can be concatenated to form a human-readable summary of the event. If multiple messages exist, they can be combined into one message. | match_only_text |
| nomad.allocation.id | Nomad allocation ID | keyword |
| nomad.namespace | Nomad namespace. | keyword |
| nomad.node.id | Nomad node ID. | keyword |
| nomad.task.name | Nomad task name. | keyword |
| related.ip | All of the IPs seen on your event. | ip |
| source.as.number | Unique number allocated to the autonomous system. The autonomous system number (ASN) uniquely identifies each network on the Internet. | long |
| source.as.organization.name | Organization name. | keyword |
| source.as.organization.name.text | Multi-field of `source.as.organization.name`. | match_only_text |
| source.geo.city_name | City name. | keyword |
| source.geo.continent_name | Name of the continent. | keyword |
| source.geo.country_iso_code | Country ISO code. | keyword |
| source.geo.country_name | Country name. | keyword |
| source.geo.location | Longitude and latitude. | geo_point |
| source.geo.region_iso_code | Region ISO code. | keyword |
| source.geo.region_name | Region name. | keyword |
| source.ip | IP address of the source (IPv4 or IPv6). | ip |
| source.port | Port of the source. | long |
| tags | List of keywords used to tag each event. | keyword |
| user.email | User email address. | keyword |
| user.id | Unique identifier of the user. | keyword |


## Operational Logs

Vault outputs its logs to stdout. In order to use the package to collect the
operational log you will need to direct its output to a file.

This table shows how the Vault field names are mapped in events. The remaining
structured data fields (indicated by the `*`) are placed under
`hashicorp_vault.log` which is mapped as `flattened` to allow for arbitrary
fields without causing mapping explosions or type conflicts.

| Original Field 	| Package Field         	|
|----------------	|-----------------------	|
| `@timestamp`   	| `@timestamp`          	|
| `@module`      	| `log.logger`          	|
| `@level`       	| `log.level`           	|
| `@message`     	| `message`             	|
| `*`            	| `hashicorp_vault.log` 	|

### Requirements

By default, Vault uses its `standard` log output as opposed to `json`. Please
enable the JSON output in order to have the log data in a structured format. In
a config file for Vault add the following:

```hcl
log_format = "json"
```

An example event for `log` looks as following:

```json
{
    "@timestamp": "2023-09-26T13:09:08.587Z",
    "agent": {
        "ephemeral_id": "5bbd86cc-8032-432d-be82-fae8f624ed98",
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "name": "docker-fleet-agent",
        "type": "filebeat",
        "version": "8.10.1"
    },
    "data_stream": {
        "dataset": "hashicorp_vault.log",
        "namespace": "ep",
        "type": "logs"
    },
    "ecs": {
        "version": "8.17.0"
    },
    "elastic_agent": {
        "id": "f25d13cd-18cc-4e73-822c-c4f849322623",
        "snapshot": false,
        "version": "8.10.1"
    },
    "event": {
        "agent_id_status": "verified",
        "dataset": "hashicorp_vault.log",
        "ingested": "2023-09-26T13:09:35Z",
        "kind": "event",
        "original": "{\"@level\":\"info\",\"@message\":\"proxy environment\",\"@timestamp\":\"2023-09-26T13:09:08.587324Z\",\"http_proxy\":\"\",\"https_proxy\":\"\",\"no_proxy\":\"\"}"
    },
    "hashicorp_vault": {
        "log": {
            "http_proxy": "",
            "https_proxy": "",
            "no_proxy": ""
        }
    },
    "host": {
        "architecture": "x86_64",
        "containerized": false,
        "hostname": "docker-fleet-agent",
        "id": "28da52b32df94b50aff67dfb8f1be3d6",
        "ip": [
            "192.168.80.5"
        ],
        "mac": [
            "02-42-C0-A8-50-05"
        ],
        "name": "docker-fleet-agent",
        "os": {
            "codename": "focal",
            "family": "debian",
            "kernel": "5.10.104-linuxkit",
            "name": "Ubuntu",
            "platform": "ubuntu",
            "type": "linux",
            "version": "20.04.6 LTS (Focal Fossa)"
        }
    },
    "input": {
        "type": "log"
    },
    "log": {
        "file": {
            "path": "/tmp/service_logs/log.json"
        },
        "level": "info",
        "offset": 709
    },
    "message": "proxy environment",
    "tags": [
        "preserve_original_event",
        "hashicorp-vault-log"
    ]
}
```

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| ecs.version | ECS version this event conforms to. `ecs.version` is a required field and must exist in all events. When querying across multiple indices -- which may conform to slightly different ECS versions -- this field lets integrations adjust to the schema version of the events. | keyword |
| event.dataset | Event dataset | constant_keyword |
| event.kind | This is one of four ECS Categorization Fields, and indicates the highest level in the ECS category hierarchy. `event.kind` gives high-level information about what type of information the event contains, without being specific to the contents of the event. For example, values of this field distinguish alert events from metric events. The value of this field can be used to inform how these kinds of events should be handled. They may warrant different retention, different access control, it may also help understand whether the data is coming in at a regular interval or not. | keyword |
| event.module | Event module | constant_keyword |
| event.original | Raw text message of entire event. Used to demonstrate log integrity or where the full log message (before splitting it up in multiple parts) may be required, e.g. for reindex. This field is not indexed and doc_values are disabled. It cannot be searched, but it can be retrieved from `_source`. If users wish to override this and index this field, please see `Field data types` in the `Elasticsearch Reference`. | keyword |
| file.path | Full path to the file, including the file name. It should include the drive letter, when appropriate. | keyword |
| file.path.text | Multi-field of `file.path`. | match_only_text |
| hashicorp_vault.log |  | flattened |
| input.type |  | keyword |
| log.file.path | Full path to the log file this event came from, including the file name. It should include the drive letter, when appropriate. If the event wasn't read from a log file, do not populate this field. | keyword |
| log.level | Original log level of the log event. If the source of the event provides a log level or textual severity, this is the one that goes in `log.level`. If your source doesn't specify one, you may put your event transport's severity here (e.g. Syslog severity). Some examples are `warn`, `err`, `i`, `informational`. | keyword |
| log.logger | The name of the logger inside an application. This is usually the name of the class which initialized the logger, or can be a custom name. | keyword |
| log.offset |  | long |
| message | For log events the message field contains the log message, optimized for viewing in a log viewer. For structured logs without an original message field, other fields can be concatenated to form a human-readable summary of the event. If multiple messages exist, they can be combined into one message. | match_only_text |
| tags | List of keywords used to tag each event. | keyword |


## Metrics

Vault can provide [telemetry](https://www.vaultproject.io/docs/configuration/telemetry)
information in the form of Prometheus metrics. You can verify that metrics are
enabled by making an HTTP request to
`http://vault_server:8200/v1/sys/metrics?format=prometheus` on your Vault server.

### Requirements

You must configure the Vault prometheus endpoint to disable the hostname
prefixing. It's recommended to also enable the hostname label.

```hcl
telemetry {
  disable_hostname = true
  enable_hostname_label = true
}
```

**Exported fields**

| Field | Description | Type | Metric Type |
|---|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |  |
| agent.id | Unique identifier of this agent (if one exists). Example: For Beats this would be beat.id. | keyword |  |
| cloud.account.id | The cloud account or organization id used to identify different entities in a multi-tenant environment. Examples: AWS account id, Google Cloud ORG Id, or other unique identifier. | keyword |  |
| cloud.availability_zone | Availability zone in which this host, resource, or service is located. | keyword |  |
| cloud.instance.id | Instance ID of the host machine. | keyword |  |
| cloud.provider | Name of the cloud provider. Example values are aws, azure, gcp, or digitalocean. | keyword |  |
| cloud.region | Region in which this host, resource, or service is located. | keyword |  |
| container.id | Unique container id. | keyword |  |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |  |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |  |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |  |
| ecs.version | ECS version this event conforms to. `ecs.version` is a required field and must exist in all events. When querying across multiple indices -- which may conform to slightly different ECS versions -- this field lets integrations adjust to the schema version of the events. | keyword |  |
| event.dataset | Event dataset | constant_keyword |  |
| event.duration | Duration of the event in nanoseconds. If `event.start` and `event.end` are known this value should be the difference between the end and start time. | long |  |
| event.kind | This is one of four ECS Categorization Fields, and indicates the highest level in the ECS category hierarchy. `event.kind` gives high-level information about what type of information the event contains, without being specific to the contents of the event. For example, values of this field distinguish alert events from metric events. The value of this field can be used to inform how these kinds of events should be handled. They may warrant different retention, different access control, it may also help understand whether the data is coming in at a regular interval or not. | keyword |  |
| event.module | Event module | constant_keyword |  |
| hashicorp_vault.metrics.\*.counter | Hashicorp Vault telemetry data from the Prometheus endpoint. | double | counter |
| hashicorp_vault.metrics.\*.histogram | Hashicorp Vault telemetry data from the Prometheus endpoint. | histogram |  |
| hashicorp_vault.metrics.\*.rate | Hashicorp Vault telemetry data from the Prometheus endpoint. | double | gauge |
| hashicorp_vault.metrics.\*.value | Hashicorp Vault telemetry data from the Prometheus endpoint. | double | gauge |
| host.name | Name of the host. It can contain what hostname returns on Unix systems, the fully qualified domain name (FQDN), or a name specified by the user. The recommended value is the lowercase FQDN of the host. | keyword |  |
| labels | Custom key/value pairs. Can be used to add meta information to events. Should not contain nested objects. All values are stored as keyword. Example: `docker` and `k8s` labels. | object |  |
| labels.auth_method | Authorization engine type. | keyword |  |
| labels.cluster | The cluster name from which the metric originated; set in the configuration file, or automatically generated when a cluster is created. | keyword |  |
| labels.creation_ttl | Time-to-live value assigned to a token or lease at creation. This value is rounded up to the next-highest bucket; the available buckets are 1m, 10m, 20m, 1h, 2h, 1d, 2d, 7d, and 30d. Any longer TTL is assigned the value +Inf. | keyword |  |
| labels.expiring |  | keyword |  |
| labels.gauge |  | keyword |  |
| labels.host |  | keyword |  |
| labels.instance |  | keyword |  |
| labels.job |  | keyword |  |
| labels.local |  | keyword |  |
| labels.mount_point | Path at which an auth method or secret engine is mounted. | keyword |  |
| labels.namespace | A namespace path, or root for the root namespace | keyword |  |
| labels.policy |  | keyword |  |
| labels.quantile |  | keyword |  |
| labels.queue_id |  | keyword |  |
| labels.term |  | keyword |  |
| labels.token_type | Identifies whether the token is a batch token or a service token. | keyword |  |
| labels.type |  | keyword |  |
| labels.version |  | keyword |  |
| service.address | Address where data about this service was collected from. This should be a URI, network address (ipv4:port or [ipv6]:port) or a resource path (sockets). | keyword |  |
| service.type | The type of the service data is collected from. The type can be used to group and correlate logs and metrics from one service type. Example: If logs or metrics are collected from Elasticsearch, `service.type` would be `elasticsearch`. | keyword |  |

