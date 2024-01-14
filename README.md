# Atomic API Design

Atomic API Design is an approach to designing APIs that enables authors to write the smallest parts of the API and use tooling to fill in the other details. It is an approach that follows the follows the [Language-Oriented Approach to API Development](https://smizell.com/language-oriented-approach/).

## How it works

1. **Write the atomic components**: Atomic components are the domain schemas and parameters of your API design. The schema components leave out technical details like URLs and HTTP methods along with non-domain properties like `id` fields and timestamps. The parameter components define filters your consumers will use to interact with collections in your API.
2. **Group the atomic components as atomic stacks**: Atomic stacks combine the atomic components into defintion that tooling will use to generate the full API design. Atomic stacks define the core schema and the filter parameters and provide configurations used by the tooling to generate the necessary OpenAPI operations.
3. **Generate the rest of the OpenAPI document**: Tooling will take the atomic stacks and generate the create, read, update, delete, and list operations in OpenAPI, all with the correct URL structure, HTTP methods, and status codes. It will generate additional schemas and parameters that aren't required to write but are needed to build the final design.

## Specification

### Atomic OpenAPI Extension (Object)

- `x-atomic` (object)
  - `stacks` (object)
    - `[stack-name]` ([Stack](#stack-object)) - The name of the atomic stack, which must be in PascalCase
   
### Stack (object)

- `schema` ([Reference](#reference-object), required) - A reference to an atomic schmea component, which MUST be a JSON Schema where the `type` is `object`. The tooling will use this to generate the additional needed schemas.
- `filters` (array of [Reference](#reference-object)) - An array of references to parameter components that will be added to the `list` operation
- `supported` (array of string), values: [`create`, `read`, `update`, `delete`, `list`] - An array of supported operations. By default, all of the operations are supported.
- `custom` (object) - Custom operations that allow for adding functionality beyond the default supported operations
  - `[custom-operation-name]` (object) - A custom operation that defines a request schema for the operation. The name MUST be in PascalCase. The response schema will match the response of the `create` operation.
    - `requestSchema` ([Reference](#reference-object)) - A reference to a schema component defining the request body JSON Schema
- `stacks` (object) - Nested stacks
  - `[stack-name]` ([Stack](#stack-object) | [Reference](#reference-object)) - The name of the atomic stack, which must be in PascalCase. It may be a [Stack](#stack-object) or a [Reference](#reference-object) to a stack.
   
### Reference (object)
   
A Reference is an OpenAPI reference, which is an object with a single property `$ref` and a JSON Pointer to the referenced object.

- `$ref` (string) - JSON Pointer to the referenced object

## Example

### Base Document

This document defines the atomic components as normal OpenAPI components. It then uses the `x-atomic` OpenAPI Extension to define the atomic stacks.

```yaml
openapi: 3.1.0
info:
  title: Example
  description: Example
  version: 1.0.0
x-atomic:
  stacks:
    Customer:
      schema:
        $ref: "#/components/schemas/Customer"
      filters:
      - $ref: "#/components/parameters/Email"
components:
  parameters:
    Email:
      in: query
      name: email
      schema:
        type: string
  schemas:
    Customer:
      type: object
      properties:
        name:
          type: string
        email:
          type: string
        active:
          type: boolean
```

### Generated document

This is a conservative definition on what a generated OpenAPI document might look like. It will generate paths and other details that match the defined API standards.

```yaml
openapi: 3.1.0
info:
  title: Example
  description: Example
  version: 1.0.0
tags:
- name: Customer
x-atomic:
  stacks:
    Customer:
      schema:
        $ref: "#/components/schemas/Customer"
      filters:
      - $ref: "#/components/parameters/Email"
# All paths are generated
paths:
  /customers:
    get:
      tags:
      - Customer
      operationId: list-customer
      responses:
        200:
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/CustomerCollection"
        400:
          description: Client Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        404:
          description: File not found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        500:
          description: Server Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
    post:
      tags:
      - Customer
      operationId: create-customer
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CustomerItem"
      responses:
        201:
          description: Created
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/CustomerItem"
        400:
          description: Client Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        404:
          description: File not found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        500:
          description: Server Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
  /customers/{customer_id}:
    parameter:
    - in: path
      name: customer_id
      required: true
      schema:
        type: string
    get:
      tags:
      - Customer
      operationId: read-customer
      parameters:
      - $ref: "#/components/parameters/Email"
      responses:
        200:
          description: OK
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/CustomerItem"
        400:
          description: Client Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        404:
          description: File not found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        500:
          description: Server Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
    put:
      tags:
      - Customer
      operationId: update-customer
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/CustomerItem"
      responses:
        200:
          description: Created
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/CustomerItem"
        400:
          description: Client Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        404:
          description: File not found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        500:
          description: Server Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
    delete:
      tags:
      - Customer
      operationId: delete-customer
      response:
        204:
          description: No Content
        400:
          description: Client Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        404:
          description: File not found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
        500:
          description: Server Error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
components:
  parameters:
    Email:
      in: query
      name: email
      schema:
        type: string
  schemas:
    Customer:
      type: object
      properties:
        name:
          type: string
        email:
          type: string
        active:
          type: boolean
    # Generated with `id`, dates, and URL
    CustomerItem:
      type: object
      properties:
        id:
          type: string
          format: uuid
          readOnly: true
        url:
          type: string
          format: url
          readOnly: true
        name:
          type: string
        email:
          type: string
        active:
          type: boolean
        createdAt:
          type: string
          format: date-time
          readOnly: true
        updatedAt:
          type: string
          format: date-time
          readOnly: true
    # Generated and references the generated CustomerItem
    CustomerCollection:
      type: object
      properties:
        url:
          type: string
          format: url
        nextUrl:
          type: string
          format: url
        previousUrl:
          type: string
          format: url
        items:
          type: array
          items:
            $ref: "#/components/schemas/CustomerItem"
    # Generated error schema, should be Problem+JSON
    Error:
      type: object
      properties:
        message:
          type: string
        details:
          type: string
```
