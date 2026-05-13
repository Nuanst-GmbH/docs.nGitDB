# Resource Model

nGitDB uses canonical resource paths instead of exposing raw file paths to application code.

## Resource Path Format

Resource paths must use:

```txt
<collection>/<id>
```

Examples:

```txt
companies/acme-gmbh
products/widget-1000
locations/berlin-office
```

Paths with more or fewer segments are rejected.

## File Resolution

By default, nGitDB maps:

```txt
<collection>/<id>
```

to:

```txt
data/<collection>/<id>/<fileName>
```

For this configuration:

```ts
{
  resourceRoot: "data",
  resources: {
    companies: {
      fileName: "company.json",
    },
  },
}
```

this resource path:

```txt
companies/acme-gmbh
```

resolves to:

```txt
data/companies/acme-gmbh/company.json
```

## JSON Documents

V1 is JSON-only. `read(resourcePath)` parses the resolved file and returns a JSON object.

```ts
const company = await db.read("companies/acme-gmbh");
```

If the file does not exist, nGitDB throws `ResourceNotFoundError`.

## Collection Definitions

Every collection must have a resource definition.

```ts
resources: {
  companies: {
    fileName: "company.json",
  },
}
```

If a resource path references an unknown collection, nGitDB throws `ResourceDefinitionError`.
