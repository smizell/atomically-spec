# Atomically

Atomically is an approach to designing APIs that enables authors to write the smallest parts of an OpenAPI document and use tooling to fill in the other details. It is an approach that follows the follows the [Language-Oriented Approach to API Development](https://smizell.com/language-oriented-approach/).

## How it works

1. **Write the atomic components**: Atomic components are the domain schemas and parameters of your API design. The schema components leave out technical details like URLs and HTTP methods along with non-domain properties like `id` fields and timestamps. The parameter components define filters your consumers will use to interact with collections in your API.
2. **Group the atomic components as atomic stacks**: Atomic stacks combine the atomic components into defintion that tooling will use to generate the full API design. Atomic stacks define the core schema and the filter parameters and provide configurations used by the tooling to generate the necessary OpenAPI operations.
3. **Generate the rest of the OpenAPI document**: Tooling will take the atomic stacks and generate the create, read, update, delete, and list operations in OpenAPI, all with the correct URL structure, HTTP methods, and status codes. It will generate additional schemas and parameters that aren't required to write but are needed to build the final design.

## Benefits and tradeoffs

The goal of this approach is to make it easier for people to learn how to design APIs and to make the API design process faster and more iterative. This Atomically Design approach accomplishes this goal by limiting the parts of the OpenAPI specification an API designer must know, along with allowing the designer to rely on tooling to make sure their API design conforms to their organization's API standards. This stands in contrast to the approaches that require designers to learn a significant portion of the OpenAPI specification along with the details for writing a conformant OpenAPI document, which often requires additional tooling like linters to make designers are following the standards correctly.

The main tradeoff of this Atomically approach is that this requires organizations to develop tooling that generates OpenAPI documents. This takes time, resources, and expertise. The hope is that this Atomically Design project can minimize these tradeoffs.

You can read more [benefits](https://smizell.com/language-oriented-approach/benefits) and [tradeoffs](https://smizell.com/language-oriented-approach/tradeoffs) in the [Language-Oriented Approach to API Development](https://smizell.com/language-oriented-approach/).

## Specification

### Atomic OpenAPI Extension (Object)

- `x-atomic` (object)
  - `version`: `"0.1.0"`
  - `stacks` (object)
    - `[stack-name]` ([Stack](#stack-object)) - The name of the atomic stack, which must be in PascalCase
   
### Stack (object)

- `schema` ([Reference](#reference-object), required) - A reference to an atomic schmea component, which MUST be a JSON Schema where the `type` is `object`. The tooling will use this to generate the additional needed schemas.
- `filters` (array of [Reference](#reference-object)) - An array of references to parameter components that will be added to the `list` operation
- `supported` (array of string), values: [`create`, `read`, `update`, `delete`, `list`] - An array of supported operations. By default, all of the operations are supported.
- `custom` (object) - Custom operations that allow for adding functionality beyond the default supported operations
  - `[custom-operation-name]` (object) - A custom operation that defines a request schema for the operation. The name MUST be in PascalCase. The response schema will match the response of the `create` operation.
    - `requestSchema` ([Reference](#reference-object)) - A reference to a schema component defining the request body JSON Schema
    - `type` values: [`collection`, `item`], default: `item` - Defines where the custom operation should be nested, either on the collection URL or the item URL
   
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
  version: 0.1.0
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
components:
  parameters:
    Email:
      in: query
      name: email
      schema:
        type: string
  schemas:
    Customer:
      properties:
        active:
          type: boolean
        email:
          type: string
        name:
          type: string
      type: object
    CustomerCollection:
      properties:
        items:
          items:
            $ref: '#/components/schemas/CustomerItem'
          type: array
        nextUrl:
          description: Next link to be used with pagination
          format: url
          type: string
      required:
      - items
      type: object
    CustomerItem:
      properties:
        active:
          type: boolean
        createdAt:
          description: Date-time the resource was created
          format: date-time
          readOnly: true
          type: string
        email:
          type: string
        id:
          description: ID of the resource
          readOnly: true
          type: string
        name:
          type: string
        updatedAt:
          description: Date-time the resource was updated
          format: date-time
          readOnly: true
          type: string
      required:
      - id
      - createdAt
      - updatedAt
      type: object
    Error:
      properties:
        detail:
          description: A human-readable explanation specific to this occurrence of
            the problem
          type: string
        instance:
          description: A URI reference that identifies the specific occurrence of
            the problem.  It may or may not yield further information if dereferenced
          type: string
        status:
          description: The HTTP status code generated by the origin server for this
            occurrence of the problem.
          type: string
        title:
          description: A short, human-readable summary of the problem type
          type: string
        type:
          description: URI reference that identifies the problem type
          type: string
      type: object
info:
  description: Example
  title: Example
  version: 1.0.0
openapi: 3.1.0
paths:
  /customers:
    get:
      operationId: list_customer
      parameters:
      - $ref: '#/components/parameters/Email'
      responses:
        '200':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CustomerCollection'
          description: OK
        '400':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Client error
        '404':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Not found
        '500':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Server error
      tags:
      - Customer
    post:
      operationId: create_customer
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CustomerItem'
      responses:
        '201':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CustomerItem'
          description: Created
        '400':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Client error
        '404':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Not found
        '500':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Server error
      tags:
      - Customer
  /customers/{customer_id}:
    delete:
      operationId: delete_customer
      parameters:
      - in: path
        name: customer_id
        required: true
        schema:
          type: string
      responses:
        '204':
          description: No content
        '400':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Client error
        '404':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Not found
        '500':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Server error
      tags:
      - Customer
    get:
      operationId: read_customer
      parameters:
      - in: path
        name: customer_id
        required: true
        schema:
          type: string
      responses:
        '200':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CustomerItem'
          description: OK
        '400':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Client error
        '404':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Not found
        '500':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Server error
      tags:
      - Customer
    put:
      operationId: update_customer
      parameters:
      - in: path
        name: customer_id
        required: true
        schema:
          type: string
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CustomerItem'
      responses:
        '200':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CustomerItem'
          description: OK
        '400':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Client error
        '404':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Not found
        '500':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
          description: Server error
      tags:
      - Customer
tags:
- name: Customer
x-atomic:
  stacks:
    Customer:
      filters:
      - $ref: '#/components/parameters/Email'
      schema:
        $ref: '#/components/schemas/Customer'
  version: 0.1.0
```
