# JSON-RPC-LD Repository Information

## Repository Details
- **Owner**: magenticmarketactualskill
- **Repository**: json-rpc-ld
- **Description**: Combining json-rpc, json-ld and SHACL
- **Languages**: TypeScript (74.8%), CSS (25.2%)
- **Commits**: 6
- **Status**: Public repository

## Repository Structure
- `.vscode/` - VS Code configuration
- `analysis/` - Analysis files
- `codegen/` - Code generation
- `knowledge/` - Knowledge base
- `site/` - Website/documentation
- `JSON-RPC-LD: A Linked Data Extension for JSON-RPC 2.0.md` - Main specification document
- `README.md` - Repository readme

## Key Technologies Combined
1. **JSON-RPC** - Remote Procedure Call protocol
2. **JSON-LD** - JSON for Linked Data
3. **SHACL** - Shapes Constraint Language (for RDF validation)

## Next Steps
- Read the main specification document: "JSON-RPC-LD: A Linked Data Extension for JSON-RPC 2.0.md"
- Explore the README.md for implementation details

## JSON-RPC-LD 0.2 Specification Summary

### Overview
JSON-RPC-LD is a lightweight extension to the JSON-RPC 2.0 protocol that incorporates semantic expressiveness of JSON-LD (JSON for Linked Data) and validation capabilities of SHACL (Shapes Constraint Language).

### Problem Statement
JSON-RPC 2.0 lacks a standardized mechanism for expressing semantic meaning of data exchanged between client and server, which can lead to ambiguity and tight coupling between implementations.

### Solution
JSON-RPC-LD introduces a mandatory mechanism for embedding JSON-LD contexts within JSON-RPC messages, allowing exchange of data that is both structured and semantically rich and machine-readable.

### Key Modifications to JSON-RPC 2.0

#### 1. The @context Member (Mandatory)
- A JSON-RPC-LD Request or Response object **MUST** include a `@context` member within the `params` or `result` object
- The `@context` value **MUST** be a valid JSON-LD context per JSON-LD 1.1 specification
- Provides mapping from terms to IRIs (Internationalized Resource Identifiers)
- Ensures all data has explicit semantic meaning and prevents ambiguity

#### 2. SHACL-based Validation (Optional)
- Servers **MAY** provide a SHACL shapes graph describing expected structure and types of `params` and `result` for each method
- Used by clients for validation and developers for documentation
- Discovery mechanism not defined in spec, but **RECOMMENDED** to provide link in API documentation

### Interoperability Note
JSON-RPC and JSON-RPC-LD are interoperable only to the degree that applications accommodate that feature. This specification does **not** allow unconstrained data types - all data types must be explicitly defined through JSON-LD contexts and validated against SHACL shapes.

### Example: getUser Method

#### Request Structure
```json
{
  "jsonrpc": "2.0",
  "method": "getUser",
  "params": {
    "@context": {
      "id": "@id",
      "User": "http://example.org/User",
      "userId": "http://example.org/User#id"
    },
    "@type": "User",
    "userId": 123
  },
  "id": 1
}
```

#### Response Structure
```json
{
  "jsonrpc": "2.0",
  "result": {
    "@context": {
      "id": "@id",
      "User": "http://example.org/User",
      "name": "http://xmlns.com/foaf/0.1/name",
      "email": "http://xmlns.com/foaf/0.1/mbox"
    },
    "@id": "http://example.org/users/123",
    "@type": "User",
    "name": "John Doe",
    "email": "john.doe@example.com"
  },
  "id": 1
}
```

#### SHACL Shapes Example
```turtle
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix ex: <http://example.org/> .

ex:GetUserParamsShape
  a sh:NodeShape ;
  sh:targetClass ex:User ;
  sh:property [
    sh:path ex:User#id ;
    sh:datatype xsd:integer ;
    sh:minCount 1 ;
    sh:maxCount 1 ;
  ] .

ex:GetUserResultShape
  a sh:NodeShape ;
  sh:targetClass ex:User ;
  sh:property [
    sh:path <http://xmlns.com/foaf/0.1/name> ;
    sh:datatype xsd:string ;
    sh:minCount 1 ;
    sh:maxCount 1 ;
  ] ;
  sh:property [
    sh:path <http://xmlns.com/foaf/0.1/mbox> ;
    sh:datatype xsd:string ;
    sh:minCount 1 ;
    sh:maxCount 1 ;
  ] .
```

### References
1. JSON-RPC 2.0 Specification
2. JSON-LD 1.1
3. Shapes Constraint Language (SHACL)
4. RFC 2119: Key words for use in RFCs to Indicate Requirement Levels

### Relevance to CodeFirst
In the CodeFirst architecture, JSON-RPC-LD is used in the **magentic_mesh** component for communication between `code-first_browser` and `code-first_server` via multiple Web Sockets. This enables semantically rich, validated communication for managing LLMs and context.
