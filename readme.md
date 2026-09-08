


# UrlMapperDemo

# UrlMapperDemo

REST URL Mapper demonstration: CRUD resources, file transfer, idempotency, FullSync API keys, BASIC and Bearer JWT authentication, and Swagger/OpenAPI documentation.

## URLs

- REST API: `http://localhost:18080/convertigo/api/v1`
- OpenAPI: `http://localhost:18080/convertigo/openapi?__project=UrlMapperDemo&JSON`

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

> The JWT below is a real short-lived demonstration token. It expires one hour after this documentation update; issue a new token with `lib_JWT.jwt_sign` when it expires.

```sh
curl --fail --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1cmxtYXBwZXItcHJvamVjdC1kb2N1bWVudGF0aW9uIiwic2NvcGUiOiJyZXNvdXJjZXM6cmVhZCIsImlhdCI6MTc4ODg4NjI0OCwiZXhwIjoxNzg4ODg5ODQ4fQ.bTAF-z3hgJPa_Op2wpNxbcYCGtGZYygSxZ90iDE174o' 'http://localhost:18080/convertigo/api/v1/resources?toto=test'
```

### Resource path parameter

```sh
curl --fail --header 'X-API-Key: c8o-demo-key-change-me' 'http://localhost:18080/convertigo/api/v1/resources/2'
```


For more technical informations : [documentation](./project.md)

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



