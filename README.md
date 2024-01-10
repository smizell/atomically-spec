# Atomic API Design

Atomic API Design is an approach to designing APIs that enables authors to write the smallest possible units and let tooling fill in the details

# Example

## Base Document

```yaml
openapi: 3.1.0
info:
  title: Example
  description: Example
  version: 1.0.0
x-atomic:
  molecules:
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

## Generated document

```yaml
openapi: 3.1.0
info:
  title: Example
  description: Example
  version: 1.0.0
tags:
- name: Customer
x-atomic:
  molecules:
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
