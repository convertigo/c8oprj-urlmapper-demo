
# ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/core/images/project_color_16x16.png?raw=true "Project") UrlMapperDemo

# UrlMapperDemo

REST URL Mapper demonstration: CRUD resources, file transfer, idempotency, FullSync API keys, BASIC and Bearer JWT authentication, and Swagger/OpenAPI documentation.

## URLs

- REST API: `http://localhost:18080/convertigo/api/v1`
- OpenAPI: `http://localhost:18080/convertigo/openapi?__project=UrlMapperDemo&JSON`

## Provision a FullSync API key

Only the SHA-256 fingerprint is persisted in FullSync. Keep the raw API key only in the client secret store and send it with `X-API-Key` or `api_key`.

1. Generate a raw key locally:

```sh
openssl rand -hex 32
```

2. Calculate its SHA-256 fingerprint locally:

```sh
printf '%s' 'paste-the-generated-raw-key-here' | shasum -a 256
```

3. In Convertigo Studio, run `UrlMapperDemo.api_keys_store.initialize_api_keys_database` only on a new or disposable demo database. This transaction resets the API key database.

4. In the Studio Test Platform, run the private `UrlMapperDemo.api_keys_store.create_api_key` transaction with the following values:

| Field | Value |
| --- | --- |
| `keyHash` | The SHA-256 output from step 2 |
| `clientId` | A stable client identifier, for example `demo-client` |
| `scopes` | Space-separated permissions, for example `resources:read resources:write files:read files:write` |
| `active` | `true` |
| `expiresAt` | A future ISO-8601 timestamp |
| `createdAt` | The current ISO-8601 timestamp |

5. Give the raw key to the API client once. It is never stored in FullSync and cannot be recovered from `keyHash`.

## Idempotency status

> Current state: `Idempotency-Key` is documented and mapped for resource creation and file upload, but persistent idempotency is not implemented yet. A repeated request can therefore still execute the business operation again.

The target implementation belongs in `api_create_resource` and `api_upload_file`, not in `auth_guard`:

1. Compute a request fingerprint from the method, path, payload, and file digest when applicable.
2. Atomically create a FullSync idempotency document keyed by `Idempotency-Key`.
3. Replay the stored response when the key and fingerprint match.
4. Return `409 Conflict` when the same key is reused with a different fingerprint.
5. Store the final status and response with an expiration policy.

## cURL examples

### OpenAPI document

```sh
curl --fail --silent 'http://localhost:18080/convertigo/openapi?__project=UrlMapperDemo&JSON'
```

### API key in the `X-API-Key` header

```sh
curl --fail --header 'X-API-Key: c8o-demo-key-change-me' 'http://localhost:18080/convertigo/api/v1/resources?toto=test'
```

### API key in the query string

```sh
curl --fail 'http://localhost:18080/convertigo/api/v1/resources?toto=test&api_key=c8o-demo-key-change-me'
```

### BASIC authentication

```sh
curl --fail --user 'openapi-basic-test:demo-password' 'http://localhost:18080/convertigo/api/v1/resources?toto=test'
```

### Bearer authentication

> The JWT below is a real ten-year demonstration token. It is for local demonstration only; do not reuse it in production.

```sh
curl --fail --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1cmxtYXBwZXItZGVtby0xMHkiLCJzY29wZSI6InJlc291cmNlczpyZWFkIHJlc291cmNlczp3cml0ZSBmaWxlczpyZWFkIGZpbGVzOndyaXRlIiwiaWF0IjoxNzg4ODg2NjY1LCJleHAiOjIxMDQ0NjI2NjV9.rjFAzSgL11nShkjuFBeWCL_nnzQMsIJG3WJoDlVyo7k' 'http://localhost:18080/convertigo/api/v1/resources?toto=test'
```

### Resource path parameter

```sh
curl --fail --header 'X-API-Key: c8o-demo-key-change-me' 'http://localhost:18080/convertigo/api/v1/resources/2'
```


<details><summary><span style="color:DarkGoldenRod"><i>References</i></span></summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/references/images/ProjectSchemaReference_16x16.png?raw=true "ProjectSchemaReference") lib_JWT

HS256 JWT validation reference.
see [readme](https://github.com/convertigo/c8oprj-lib-jwt/tree/master#readme)
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Connectors</i></span></summary><blockquote><p>


<details><summary><b>api_keys_store</b> : Internal API key repository: fingerprints, scopes, status, expiration, and audit information</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/connectors/images/fullsyncconnector_color_16x16.png?raw=true "FullSyncConnector") api_keys_store

Internal API key repository: fingerprints, scopes, status, expiration, and audit information. Never store a raw key here.

<details><summary><span style="color:DarkGoldenRod"><i>Transactions</i></span></summary><blockquote><p>


<details><summary><b>create_api_key</b> : Internal API key provisioning: stores keyHash only, never the raw key</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/couchdb/images/postdocument_color_16x16.png?raw=true "PostDocumentTransaction") create_api_key

Internal API key provisioning: stores keyHash only, never the raw key.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;active
</td>
<td>
API key active/inactive status.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;clientId
</td>
<td>
Owning client identifier.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;createdAt
</td>
<td>
ISO-8601 creation timestamp.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;expiresAt
</td>
<td>
ISO-8601 expiration timestamp.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;keyHash
</td>
<td>
SHA-256 fingerprint; never the raw key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;scopes
</td>
<td>
List of authorized scopes.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>find_api_key_by_hash</b> : Internal lookup of an active API key by SHA-256 fingerprint</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/couchdb/images/postfind_color_16x16.png?raw=true "PostFindTransaction") find_api_key_by_hash

Internal lookup of an active API key by SHA-256 fingerprint. The raw key must never be sent to FullSync.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;keyHash
</td>
<td>
SHA-256 fingerprint of the supplied API key.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>initialize_api_keys_database</b> : Initializes the dedicated API key database</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/couchdb/images/resetdatabase_color_16x16.png?raw=true "ResetDatabaseTransaction") initialize_api_keys_database

Initializes the dedicated API key database. Do not run this against a production database containing keys.
</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>

<details><summary><b>UrlMapperDemodb</b> : HSQLDB CRUD data source for UrlMapperDemo</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/connectors/images/sqlconnector_color_16x16.png?raw=true "SqlConnector") UrlMapperDemodb

HSQLDB CRUD data source for UrlMapperDemo.

<details><summary><span style="color:DarkGoldenRod"><i>Transactions</i></span></summary><blockquote><p>


<details><summary><b>count_resources</b> : Count Resource rows</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/images/sqltransaction_color_16x16.png?raw=true "SqlTransaction") count_resources

Count Resource rows.
</p></blockquote></details>

<details><summary><b>create_resource</b> : Create one resource row</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/images/sqltransaction_color_16x16.png?raw=true "SqlTransaction") create_resource

Create one resource row.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;description
</td>
<td>
description for resource.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;name
</td>
<td>
name for resource.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;version
</td>
<td>
version for resource.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>delete_resource</b> : Delete one resource row by primary key</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/images/sqltransaction_color_16x16.png?raw=true "SqlTransaction") delete_resource

Delete one resource row by primary key.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;id
</td>
<td>
resource primary key.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>init_schema</b> : Create or update the SQL schema for Resource and seed demo rows</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/images/sqltransaction_color_16x16.png?raw=true "SqlTransaction") init_schema

Create or update the SQL schema for Resource and seed demo rows.
</p></blockquote></details>

<details><summary><b>list_resources</b> : List Resource rows</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/images/sqltransaction_color_16x16.png?raw=true "SqlTransaction") list_resources

List Resource rows.
</p></blockquote></details>

<details><summary><b>read_resource</b> : Read one resource row by primary key</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/images/sqltransaction_color_16x16.png?raw=true "SqlTransaction") read_resource

Read one resource row by primary key.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;id
</td>
<td>
resource primary key.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>update_resource</b> : Update one resource row by primary key</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/transactions/images/sqltransaction_color_16x16.png?raw=true "SqlTransaction") update_resource

Update one resource row by primary key.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;description
</td>
<td>
description for resource.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;id
</td>
<td>
resource primary key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;name
</td>
<td>
name for resource.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;version
</td>
<td>
version for resource.
</td>
</tr>
</table>

</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Sequences</i></span></summary><blockquote><p>


<details><summary><b>api_count_resources</b> : Protected count: auth_guard returns a contract that is explicitly checked before business processing</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") api_count_resources

Protected count: auth_guard returns a contract that is explicitly checked before business processing. Scope: resources:read.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
API key received from X-API-Key or api_key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;token
</td>
<td>
Bearer JWT passed to the guard.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>api_create_resource</b> : Protected creation: auth_guard returns a contract that is explicitly checked before business processing</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") api_create_resource

Protected creation: auth_guard returns a contract that is explicitly checked before business processing. Scope: resources:write.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
API key passed to auth_guard.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;description
</td>
<td>
description for resource.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;idempotencyKey
</td>
<td>
Idempotency key. Persist it with the request fingerprint and response.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;name
</td>
<td>
name for resource.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;token
</td>
<td>
Bearer JWT passed to auth_guard.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;version
</td>
<td>
version for resource.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>api_delete_resource</b> : Protected deletion: auth_guard returns a contract that is explicitly checked before business processing</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") api_delete_resource

Protected deletion: auth_guard returns a contract that is explicitly checked before business processing. Scope: resources:write.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
API key passed to auth_guard.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;id
</td>
<td>
resource primary key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;token
</td>
<td>
Bearer JWT passed to auth_guard.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>api_download_file</b> : Protected download: auth_guard returns a contract that is explicitly checked before business processing</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") api_download_file

Protected download: auth_guard returns a contract that is explicitly checked before business processing. Scope: files:read.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
API key passed to auth_guard.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;id
</td>
<td>
Identifier of the resource whose file is to be retrieved.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;token
</td>
<td>
Bearer JWT passed to auth_guard.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>api_list_resources</b> : Protected list: auth_guard returns a contract that is explicitly checked before business processing</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") api_list_resources

Protected list: auth_guard returns a contract that is explicitly checked before business processing. Scope: resources:read.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
API key received from X-API-Key or api_key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKeyHeader
</td>
<td>
API key passed through X-API-Key. It must be checked by the guard sequence.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKeyUrl
</td>
<td>
API key passed through api_key. Not recommended outside demonstrations.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;authHeader
</td>
<td>
Bearer JWT passed through Authorization. It must be validated by a guard sequence.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;token
</td>
<td>
Bearer JWT passed to auth_guard.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;toto
</td>
<td>
Demonstration filter received through ?toto=test.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>api_read_resource</b> : Protected read: auth_guard returns a contract that is explicitly checked before business processing</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") api_read_resource

Protected read: auth_guard returns a contract that is explicitly checked before business processing. Scope: resources:read.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
API key passed to auth_guard.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;id
</td>
<td>
resource primary key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;token
</td>
<td>
Bearer JWT passed to auth_guard.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>api_update_resource</b> : Protected update: auth_guard returns a contract that is explicitly checked before business processing</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") api_update_resource

Protected update: auth_guard returns a contract that is explicitly checked before business processing. Scope: resources:write.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
API key passed to auth_guard.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;description
</td>
<td>
description for resource.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;id
</td>
<td>
resource primary key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;name
</td>
<td>
name for resource.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;token
</td>
<td>
Bearer JWT passed to auth_guard.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;version
</td>
<td>
version for resource.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>api_upload_file</b> : Protected upload: auth_guard returns a contract that is explicitly checked before business processing</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") api_upload_file

Protected upload: auth_guard returns a contract that is explicitly checked before business processing. Scope: files:write.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
API key passed to auth_guard.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;file
</td>
<td>
Uploaded file (multipart/form-data).
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;id
</td>
<td>
Target resource identifier.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;idempotencyKey
</td>
<td>
Upload idempotency key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;token
</td>
<td>
Bearer JWT passed to auth_guard.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>auth_guard</b> : Unified FullSync API key or Bearer JWT guard</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") auth_guard

Unified FullSync API key or Bearer JWT guard. Returns authorized, httpStatus, and authError; the facade explicitly checks this contract.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;apiKey
</td>
<td>
Optional API key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;requiredScope
</td>
<td>
API scope required by the calling operation.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;token
</td>
<td>
Optional Bearer JWT.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>auth_login</b> : Validates the demonstration BASIC credential before marking the current session as authenticated</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") auth_login

Validates the demonstration BASIC credential before marking the current session as authenticated.

<span style="color:DarkGoldenRod">Variables</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;password
</td>
<td>
BASIC password to validate.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/variables/images/variable_color_16x16.png?raw=true "  alt="RequestableVariable" >&nbsp;user
</td>
<td>
BASIC username to validate.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>auth_logout</b> : Authentication skeleton</summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/sequences/images/genericsequence_color_16x16.png?raw=true "GenericSequence") auth_logout

Authentication skeleton. Clears the current session authenticated user.
</p></blockquote></details>
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Rest Web Service</i></span></summary><blockquote><p>


## ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/core/images/urlmapper_color_16x16.png?raw=true "UrlMapper") ApiV1

REST API demonstration: Swagger/OpenAPI, CRUD, file handling, idempotency, and security.

<details><summary><span style="color:DarkGoldenRod"><i>Authentications</i></span></summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/basicauthentication_color_16x16.png?raw=true "BasicAuthentication") Basic

Declares BASIC authentication in the generated OpenAPI definition; the guard validates it for every REST operation.
</p></blockquote></details>

<details><summary><span style="color:DarkGoldenRod"><i>Mappings</i></span></summary><blockquote><p>


<details><summary><b>/resources</b> : Resource collection</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/pathmapping_color_16x16.png?raw=true "PathMapping") /resources

Resource collection. Example query parameter: ?toto=test.

<details><summary><span style="color:DarkGoldenRod"><i>Operations</i></span></summary><blockquote><p>


<details><summary><b>CreateResource</b> : Creates a resource; the Idempotency-Key is documented</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/postoperation_color_16x16.png?raw=true "PostOperation") CreateResource

Creates a resource; the Idempotency-Key is documented.

<span style="color:DarkGoldenRod">Parameters</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;Authorization
</td>
<td>
JWT Bearer : Authorization: Bearer <token>.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;Idempotency_Key
</td>
<td>
Idempotency key: send the standard Idempotency-Key header. The Idempotency_Key mapper alias is also accepted.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;X_API_Key
</td>
<td>
API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Responses</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/operationresponse_color_16x16.png?raw=true "  alt="OperationResponse" >&nbsp;201-Response
</td>
<td>
Resource created or response replayed for the same idempotency key.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/operationresponse_color_16x16.png?raw=true "  alt="OperationResponse" >&nbsp;409-Response
</td>
<td>
Idempotency key reused with a different payload.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>ListResources</b> : Resource list; the toto demonstration filter is available</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/getoperation_color_16x16.png?raw=true "GetOperation") ListResources

Resource list; the toto demonstration filter is available.

<span style="color:DarkGoldenRod">Parameters</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/queryparameter_color_16x16.png?raw=true "  alt="QueryParameter" >&nbsp;api_key
</td>
<td>
API key in the URL: ?api_key=... (not recommended outside demonstrations).
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;Authorization
</td>
<td>
JWT Bearer : Authorization: Bearer <token>.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/queryparameter_color_16x16.png?raw=true "  alt="QueryParameter" >&nbsp;toto
</td>
<td>
Demonstration filter, for example ?toto=test.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;X_API_Key
</td>
<td>
API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Responses</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/operationresponse_color_16x16.png?raw=true "  alt="OperationResponse" >&nbsp;200-Response
</td>
<td>
Resource list.
</td>
</tr>
</table>

</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>

<details><summary><b>/resources/{id}</b> : Resource identified by the {id} path parameter</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/pathmapping_color_16x16.png?raw=true "PathMapping") /resources/{id}

Resource identified by the {id} path parameter.

<details><summary><span style="color:DarkGoldenRod"><i>Operations</i></span></summary><blockquote><p>


<details><summary><b>DeleteResource</b> : Deletes a resource by ID</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/deleteoperation_color_16x16.png?raw=true "DeleteOperation") DeleteResource

Deletes a resource by ID.

<span style="color:DarkGoldenRod">Parameters</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;Authorization
</td>
<td>
JWT Bearer : Authorization: Bearer <token>.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/pathparameter_color_16x16.png?raw=true "  alt="PathParameter" >&nbsp;id
</td>
<td>
Integer resource identifier, for example /resources/2.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;X_API_Key
</td>
<td>
API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>ReadResource</b> : Reads a resource by ID</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/getoperation_color_16x16.png?raw=true "GetOperation") ReadResource

Reads a resource by ID.

<span style="color:DarkGoldenRod">Parameters</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;Authorization
</td>
<td>
JWT Bearer : Authorization: Bearer <token>.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/pathparameter_color_16x16.png?raw=true "  alt="PathParameter" >&nbsp;id
</td>
<td>
Integer resource identifier, for example /resources/2.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;X_API_Key
</td>
<td>
API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Responses</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/operationresponse_color_16x16.png?raw=true "  alt="OperationResponse" >&nbsp;200-Response
</td>
<td>
Resource found.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/operationresponse_color_16x16.png?raw=true "  alt="OperationResponse" >&nbsp;404-Response
</td>
<td>
Resource not found.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>UpdateResource</b> : Updates a resource by ID</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/putoperation_color_16x16.png?raw=true "PutOperation") UpdateResource

Updates a resource by ID.

<span style="color:DarkGoldenRod">Parameters</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;Authorization
</td>
<td>
JWT Bearer : Authorization: Bearer <token>.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/pathparameter_color_16x16.png?raw=true "  alt="PathParameter" >&nbsp;id
</td>
<td>
Integer resource identifier, for example /resources/2.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;X_API_Key
</td>
<td>
API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.
</td>
</tr>
</table>

</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>

<details><summary><b>/resources/{id}/file</b> : Upload and retrieval of the file associated with a resource</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/pathmapping_color_16x16.png?raw=true "PathMapping") /resources/{id}/file

Upload and retrieval of the file associated with a resource.

<details><summary><span style="color:DarkGoldenRod"><i>Operations</i></span></summary><blockquote><p>


<details><summary><b>DownloadFile</b> : Returns the binary content of the resource file</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/getoperation_color_16x16.png?raw=true "GetOperation") DownloadFile

Returns the binary content of the resource file.

<span style="color:DarkGoldenRod">Parameters</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;Authorization
</td>
<td>
JWT Bearer : Authorization: Bearer <token>.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/pathparameter_color_16x16.png?raw=true "  alt="PathParameter" >&nbsp;id
</td>
<td>
Integer resource identifier, for example /resources/2.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;X_API_Key
</td>
<td>
API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Responses</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/operationresponse_color_16x16.png?raw=true "  alt="OperationResponse" >&nbsp;404-Response
</td>
<td>
File not found.
</td>
</tr>
</table>

</p></blockquote></details>

<details><summary><b>UploadFile</b> : Multipart/form-data: adds or replaces a resource file</summary><blockquote><p>


### ![](https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/postoperation_color_16x16.png?raw=true "PostOperation") UploadFile

Multipart/form-data: adds or replaces a resource file. The Idempotency-Key header is recommended.

<span style="color:DarkGoldenRod">Parameters</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;Authorization
</td>
<td>
JWT Bearer : Authorization: Bearer <token>.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/pathparameter_color_16x16.png?raw=true "  alt="PathParameter" >&nbsp;id
</td>
<td>
Integer resource identifier, for example /resources/2.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;Idempotency_Key
</td>
<td>
Upload idempotency key: send the standard Idempotency-Key header. The Idempotency_Key mapper alias is also accepted.
</td>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/headerparameter_color_16x16.png?raw=true "  alt="HeaderParameter" >&nbsp;X_API_Key
</td>
<td>
API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.
</td>
</tr>
</table>


<span style="color:DarkGoldenRod">Responses</span>

<table>
<tr>
<th>
name
</th>
<th>
comment
</th>
</tr>
<tr>
<td>
<img src="https://github.com/convertigo/convertigo/blob/develop/engine/src/com/twinsoft/convertigo/beans/rest/images/operationresponse_color_16x16.png?raw=true "  alt="OperationResponse" >&nbsp;201-Response
</td>
<td>
File accepted.
</td>
</tr>
</table>

</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>
</p></blockquote></details>
