


# UrlMapperDemo

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



For more technical informations : [documentation](./project.md)

- [Installation](#installation)
- [Rest Web Service](#rest-web-service)
    - [Mappings](#mappings)
        - [/resources](#resources)
            - [Operations](#operations)
                - [CreateResource](#createresource)
                - [ListResources](#listresources)
        - [/resources/{id}](#resources{id})
            - [Operations](#operations-1)
                - [DeleteResource](#deleteresource)
                - [ReadResource](#readresource)
                - [UpdateResource](#updateresource)
        - [/resources/{id}/file](#resources{id}file)
            - [Operations](#operations-2)
                - [DownloadFile](#downloadfile)
                - [UploadFile](#uploadfile)


## Installation

1. In your Convertigo Studio click on ![](https://github.com/convertigo/convertigo/blob/develop/eclipse-plugin-studio/icons/studio/project_import.gif?raw=true "Import a project in treeview") to import a project in the treeview
2. In the import wizard

   ![](https://github.com/convertigo/convertigo/blob/develop/eclipse-plugin-studio/tomcat/webapps/convertigo/templates/ftl/project_import_wzd.png?raw=true "Import Project")
   
   paste the text below into the `Project remote URL` field:
   <table>
     <tr><td>Usage</td><td>Click the copy button at the end of the line</td></tr>
     <tr><td>To contribute</td><td>

     ```
     UrlMapperDemo=https://github.com/convertigo/c8oprj-urlmapper-demo.git:branch=master
     ```
     </td></tr>
     <tr><td>To simply use</td><td>

     ```
     UrlMapperDemo=https://github.com/convertigo/c8oprj-urlmapper-demo/archive/master.zip
     ```
     </td></tr>
    </table>
3. Click the `Finish` button. This will automatically import the __UrlMapperDemo__ project


## Rest Web Service

REST API demonstration: Swagger/OpenAPI, CRUD, file handling, idempotency, and security.

### Mappings

#### /resources

Resource collection. Example query parameter: ?toto=test.

##### Operations

###### CreateResource

Creates a resource; the Idempotency-Key is documented.

**Parameters**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>Authorization</td><td>JWT Bearer : Authorization: Bearer <token>.</td>
</tr>
<tr>
<td>Idempotency_Key</td><td>Idempotency key: send the standard Idempotency-Key header. The Idempotency_Key mapper alias is also accepted.</td>
</tr>
<tr>
<td>X_API_Key</td><td>API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.</td>
</tr>
</table>

###### ListResources

Resource list; the toto demonstration filter is available.

**Parameters**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>api_key</td><td>API key in the URL: ?api_key=... (not recommended outside demonstrations).</td>
</tr>
<tr>
<td>Authorization</td><td>JWT Bearer : Authorization: Bearer <token>.</td>
</tr>
<tr>
<td>toto</td><td>Demonstration filter, for example ?toto=test.</td>
</tr>
<tr>
<td>X_API_Key</td><td>API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.</td>
</tr>
</table>

#### /resources/{id}

Resource identified by the {id} path parameter.

##### Operations

###### DeleteResource

Deletes a resource by ID.

**Parameters**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>Authorization</td><td>JWT Bearer : Authorization: Bearer <token>.</td>
</tr>
<tr>
<td>id</td><td>Integer resource identifier, for example /resources/2.</td>
</tr>
<tr>
<td>X_API_Key</td><td>API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.</td>
</tr>
</table>

###### ReadResource

Reads a resource by ID.

**Parameters**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>Authorization</td><td>JWT Bearer : Authorization: Bearer <token>.</td>
</tr>
<tr>
<td>id</td><td>Integer resource identifier, for example /resources/2.</td>
</tr>
<tr>
<td>X_API_Key</td><td>API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.</td>
</tr>
</table>

###### UpdateResource

Updates a resource by ID.

**Parameters**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>Authorization</td><td>JWT Bearer : Authorization: Bearer <token>.</td>
</tr>
<tr>
<td>id</td><td>Integer resource identifier, for example /resources/2.</td>
</tr>
<tr>
<td>X_API_Key</td><td>API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.</td>
</tr>
</table>

#### /resources/{id}/file

Upload and retrieval of the file associated with a resource.

##### Operations

###### DownloadFile

Returns the binary content of the resource file.

**Parameters**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>Authorization</td><td>JWT Bearer : Authorization: Bearer <token>.</td>
</tr>
<tr>
<td>id</td><td>Integer resource identifier, for example /resources/2.</td>
</tr>
<tr>
<td>X_API_Key</td><td>API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.</td>
</tr>
</table>

###### UploadFile

Multipart/form-data: adds or replaces a resource file. The Idempotency-Key header is recommended.

**Parameters**

<table>
<tr>
<th>name</th><th>comment</th>
</tr>
<tr>
<td>Authorization</td><td>JWT Bearer : Authorization: Bearer <token>.</td>
</tr>
<tr>
<td>id</td><td>Integer resource identifier, for example /resources/2.</td>
</tr>
<tr>
<td>Idempotency_Key</td><td>Upload idempotency key: send the standard Idempotency-Key header. The Idempotency_Key mapper alias is also accepted.</td>
</tr>
<tr>
<td>X_API_Key</td><td>API key: send the standard X-API-Key header. The X_API_Key mapper alias is also accepted; valid BASIC or Bearer authentication is an alternative.</td>
</tr>
</table>



