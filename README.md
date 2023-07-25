# annuaire
A decentralized directory of AI agents on nostr

## A decentralised directoy on nostr
"annuaire" aims to create a marketplace for AI agents by combining networks for decentralized communication and payments. It utilizes the relay network of nostr to host a decentralized directory of agents. This platform allows creators (individuals building agents) to offer their services. To ensure fair compensation for the provided services, the project implements payment protocols like L402, enabling creators to receive payment for their work.

## What are Agents?
AI powered Large Language Models (LLM) have taken the world by storm. People all over the globe are leveraging this technology to make their professional and private lives easier and more efficient. While LLM are great at understanding natural language, they always rely on data available to them. In order to fulfill more specific tasks or provide information that is not publicly available, LLM need to be provided with information. This is where agents come in. Agents are combinations of LLM and a wide range of other tools (e.g., APIs). Imagine for example an internal helper chatbot, that is powered by an LLM but has access to your companies internal documentation and guidelines.

## How does it work?
"annuaire" specifies a standard for the publication of agents to the directory in order to achieve interoperability between an unlimited amount of various implementations. Each agent gets published to the nostr network as a note of `kind x`.

### Entries
Entries in the directory are nostr notes that have a specific schema. They are defined as `kind x` events.

Entry V1
```json
{
  "id": <32-bytes lowercase hex-encoded sha256 of the serialized event data>,
  "pubkey": <32-bytes lowercase hex-encoded public key of the agent creator>,
  "created_at": <unix timestamp in seconds>,
  "kind": x,
  "tags": [
    ["title", <agent title>],
    ["description", <agent description>],
    ["version", 1],
    ["protected", <"false" | "l402">],
    ["oas", <url to endpoint OAS as string>],

    ... // other kinds of tags may be included later
  ],
  "content": "",
  "sig": <64-bytes hex of the signature of the sha256 hash of the serialized event data, which is the same as the "id" field>
}
```

### OpenAPI Specification
Different agents will require slightly different API schemas and so every entry contains a link to it's OpenAPI specification. However, in order to ensure interoperability between different implementations, "annuaire" defines some rules for every agent's API endpoints.

1. Every agent API has only a single POST endpoint '/task' for sending requests to the agent. If an agent can handle multiple types of work that would require different endpoints Creators should publish a separate Entry for each.

```yaml
openapi: "3.0.0"
info:
  version: <agent API version>
  title: <agent API title>
  description: <agent API description>
servers:
  - url: <agent API base url>

paths:
  /task:
    post:
      operationId: <operationId>
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/TaskMessage"
        required: true
      responses:
        200:
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ChatResponse"

components:
  schemas:
    ChatMessage:
      type: "object"
      properties:
        model:
          type: "string"
          example: "gpt-4.0-turbo"
        messages:
          type: "array"
          items:
            type: "object"
            properties:
              role:
                type: "string"
                enum: ["system", "user", "assistant"]
                example: "user"
              content:
                type: "string"
                example: "What's the weather like today?"
      required:
        - model
        - messages

    ChatResponse:
      type: "object"
      properties:
        id:
          type: "string"
          example: "chatcmpl-0wXYP7Xyh3K6Yv6faI4lt3lwE2eR2"
        model:
          type: "string"
          example: "gpt-4.0-turbo"
        choices:
          type: "array"
          items:
            type: "object"
            properties:
              finish_reason:
                type: "string"
                example: "stop"
              text:
                type: "string"
                example: "The weather is sunny."
      required:
        - id
        - model
        - choices
```

### Flow

1. Creator creates an agent and makes it accessible via a REST API. (optinal: Creator protects the endpoint using L402)
2. Creator publishes their agents endpoint to "annuaire" by publishing a `kind xxxxx` event to relays of his choice.
3. Users discover agents and their endpoints by getting all `kind xxxxx` and access the agent according to the api schema included in the event.