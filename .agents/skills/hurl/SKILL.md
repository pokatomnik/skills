---
name: generate-hurl-request
description: Generate simple Hurl request files from OpenAPI operations. Use when the user wants an executable HTTP request similar to curl, without response assertions, captures, chained scenarios, or test logic.
---

# Generate Hurl Request

Generate small, readable `.hurl` files from OpenAPI operations.

The result should behave as a convenient, reusable alternative to a `curl` command.

## Scope

Generate only HTTP requests.

Do not generate:

- `HTTP 200` or any other response section
- `[Asserts]`
- `[Captures]`
- response headers
- expected response bodies
- chained request scenarios
- polling or retry logic
- cleanup requests
- performance tests
- test reports

By default, generate one independent request per `.hurl` file.

## Main Rules

1. Use only endpoints, methods, parameters, body fields, content types, and authentication schemes present in the OpenAPI specification.
2. Do not invent undocumented fields or values.
3. Prefer variables for hostnames, credentials, identifiers, and values likely to change.
4. Never hardcode real secrets.
5. Use examples from OpenAPI when available.
6. When required information is missing, use a variable or add a `# TODO:` comment.
7. Keep the generated Hurl file minimal. Do not add headers such as `User-Agent`, `Accept`, or `Content-Type` unless they are useful or required.
8. The request body must be the last part of the request.

## Basic Hurl Request

```hurl
GET {{base_url}}/pets
```

Run it with:

```shell
hurl --variable base_url=http://localhost:8080 request.hurl
```

## Variables

Use Hurl templates:

```hurl
GET {{base_url}}/pets/{{pet_id}}
```

Variables can be passed with command-line options:

```shell
hurl \
  --variable base_url=http://localhost:8080 \
  --variable pet_id=42 \
  request.hurl
```

They can also be loaded from a variables file:

```shell
hurl --variables-file vars.env request.hurl
```

Example `vars.env`:

```dotenv
base_url=http://localhost:8080
pet_id=42
access_token=replace-me
```

Variables can also be provided through environment variables:

```shell
export HURL_VARIABLE_base_url=http://localhost:8080
export HURL_VARIABLE_pet_id=42

hurl request.hurl
```

Use secrets for sensitive values when possible:

```shell
hurl \
  --variable base_url=https://api.example.com \
  --secret access_token="$ACCESS_TOKEN" \
  request.hurl
```

Inside the Hurl file:

```hurl
GET {{base_url}}/account
Authorization: Bearer {{access_token}}
```

## OpenAPI Mapping

| OpenAPI element | Hurl |
|---|---|
| HTTP method | `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, and so on |
| Server URL | `{{base_url}}` |
| Path parameter | `{{parameter_name}}` in URL |
| Query parameter | `[Query]` |
| Header parameter | Header directly below URL |
| Cookie parameter | `[Cookies]` |
| Basic authentication | `[BasicAuth]` |
| Bearer token | `Authorization: Bearer {{access_token}}` |
| Header API key | Header with a variable |
| Query API key | `[Query]` with a variable |
| JSON request body | Inline JSON |
| URL-encoded body | `[Form]` |
| Multipart body | `[Multipart]` |
| Binary body | `file,path;` |

## Path Parameters

Convert OpenAPI path placeholders into Hurl variables.

OpenAPI path:

```text
/pets/{petId}
```

Hurl:

```hurl
GET {{base_url}}/pets/{{pet_id}}
```

Do not leave OpenAPI placeholders unchanged:

```hurl
# Wrong
GET {{base_url}}/pets/{petId}
```

## Query Parameters

Prefer `[Query]` instead of manually building the query string.

```hurl
GET {{base_url}}/pets
[Query]
limit: {{limit}}
status: available
```

Hurl encodes query parameter values.

Do not pre-encode ordinary values unless the OpenAPI contract explicitly requires encoded input.

When OpenAPI uses unusual serialization such as `deepObject`, `matrix`, `label`, `spaceDelimited`, or `pipeDelimited`, preserve the required wire format. If it cannot be derived safely, add a TODO instead of guessing.

## Headers

Headers must appear directly after the method and URL.

```hurl
GET {{base_url}}/reports
Authorization: Bearer {{access_token}}
X-Tenant-Id: {{tenant_id}}
Accept: application/json
[Query]
page: {{page}}
```

Do not put headers after `[Query]`, `[Form]`, `[Multipart]`, `[Cookies]`, or another section.

Wrong:

```hurl
GET {{base_url}}/reports
[Query]
page: 1
X-Tenant-Id: {{tenant_id}}
```

## Authentication

### Basic authentication

```hurl
GET {{base_url}}/admin
[BasicAuth]
{{username}}: {{password}}
```

### Bearer token

```hurl
GET {{base_url}}/account
Authorization: Bearer {{access_token}}
```

### API key in header

```hurl
GET {{base_url}}/account
X-API-Key: {{api_key}}
```

### API key in query

```hurl
GET {{base_url}}/account
[Query]
api_key: {{api_key}}
```

### API key in cookie

```hurl
GET {{base_url}}/account
[Cookies]
api_key: {{api_key}}
```

Respect OpenAPI security semantics:

- operation-level `security` overrides root-level `security`
- entries in the `security` array are alternatives
- multiple schemes in one security object are all required
- an empty `security` array means no authentication

Choose one supported alternative unless the user requests a specific one.

## JSON Request Body

Use inline JSON:

```hurl
POST {{base_url}}/pets
Accept: application/json
{
  "name": "{{pet_name}}",
  "age": {{pet_age}},
  "vaccinated": {{vaccinated}}
}
```

Hurl automatically treats an inline JSON body as JSON.

Preserve JSON types:

```hurl
{
  "name": "{{name}}",
  "count": {{count}},
  "enabled": {{enabled}},
  "value": {{nullable_value}}
}
```

Use quotes for strings.

Do not quote numbers, booleans, objects, arrays, or `null`.

For a vendor-specific JSON media type, set it explicitly:

```hurl
POST {{base_url}}/pets
Content-Type: application/vnd.example.pet+json
{
  "name": "{{pet_name}}"
}
```

## Form Body

For `application/x-www-form-urlencoded`:

```hurl
POST {{base_url}}/login
[Form]
username: {{username}}
password: {{password}}
```

## Multipart Body

For `multipart/form-data`:

```hurl
POST {{base_url}}/avatars
Authorization: Bearer {{access_token}}
[Multipart]
description: {{description}}
file: file,fixtures/avatar.png; image/png
```

Do not invent fixture files.

When a file path is unknown:

```hurl
# TODO: provide an existing upload fixture
file: file,fixtures/example.bin; application/octet-stream
```

## Text Body

Single-line text:

```hurl
POST {{base_url}}/echo
Content-Type: text/plain
`Hello world`
```

Multiline text:

~~~hurl
POST {{base_url}}/documents
Content-Type: text/plain
```
line one
line two
```
~~~

## Binary Body

```hurl
PUT {{base_url}}/objects/{{object_key}}
Content-Type: application/octet-stream
file,fixtures/payload.bin;
```

The body must be the last part of the request.

## Selecting Values from OpenAPI

Use this priority:

1. parameter or media type `example`
2. an entry from `examples`
3. schema `example`
4. schema `default`
5. first allowed `enum` value
6. a simple deterministic valid value
7. a variable
8. `# TODO:` when a valid value cannot be determined

Respect:

- required fields
- `minimum` and `maximum`
- string length limits
- patterns
- enum values
- array size limits
- `readOnly`
- `writeOnly`

For request bodies:

- include required properties
- omit `readOnly: true` properties
- include optional properties only when useful
- include `writeOnly: true` properties when required

## Schema Composition

For `allOf`, combine applicable request properties and constraints.

For `oneOf` or `anyOf`, choose one valid branch using:

1. a documented example
2. discriminator mapping
3. the smallest clearly valid branch

Do not merge all `oneOf` branches.

If a valid branch cannot be selected safely, use variables or add a TODO.

## Multiple Content Types

Choose one request content type.

Default preference when the user gives no preference:

1. `application/json`
2. vendor `+json`
3. `application/x-www-form-urlencoded`
4. `multipart/form-data`
5. XML
6. text
7. binary

Do not combine several request content types into one Hurl file.

## Output Format

When creating files, use descriptive names:

```text
get_pet.hurl
create_pet.hurl
upload_avatar.hurl
```

When responding in chat, return:

1. the filename
2. one fenced `hurl` block
3. an optional `vars.env.example` block

Do not include explanations inside the Hurl code block except useful comments and TODOs.

## Validation

Format the generated file:

```shell
hurlfmt --in-place request.hurl
```

Check that it parses and is already formatted:

```shell
hurlfmt --check request.hurl
```

Run the request:

```shell
hurl --variables-file vars.env request.hurl
```

For debugging:

```shell
hurl --very-verbose --variables-file vars.env request.hurl
```

## Final Checklist

Before returning the request, verify:

- the endpoint exists in OpenAPI
- the HTTP method is correct
- every path parameter uses `{{variable}}`
- required query, header, cookie, and body values are present
- authentication matches the effective OpenAPI security rules
- headers appear before request sections
- the body is last
- JSON variable types are quoted correctly
- no secrets are hardcoded
- no response section is present
- no assertions or captures are present
- no chained requests are present
- fixture paths exist or contain a TODO
- the result is valid Hurl syntax
