# CodeFirst UML Design Documentation

## Overview

This document provides comprehensive UML diagrams illustrating the CodeFirst architecture, components, and typical use cases. CodeFirst enables **magnetic (man/agent) application deployment** by seamlessly integrating human users and AI agents through two primary mesh systems: **application_mesh** (using Wayland) and **magentic_mesh** (using JSON-RPC-LD).

---

## Diagram Index

### Structural Diagrams

1. **Component Diagram** - Shows the major components and their relationships
2. **Class Diagram** - Details the core classes and their interactions
3. **Deployment Diagram** - Illustrates how components are deployed across infrastructure

### Behavioral Diagrams

4. **Use Case Diagram** - Depicts system use cases for different actors
5. **Sequence Diagram: Input Event Handling** - Shows user input processing via application_mesh
6. **Sequence Diagram: LLM Interaction** - Demonstrates AI agent communication via magentic_mesh
7. **Sequence Diagram: Complete Flow** - Illustrates end-to-end magnetic application interaction

---

## 1. Component Diagram

### Purpose
Illustrates the high-level architecture of CodeFirst, showing how the two mesh systems (application_mesh and magentic_mesh) work together, along with language implementations and legacy framework support.

### Key Components

**application_mesh**
- **code-first_browser**: Wayland Compositor running in the browser
  - Manages multiple application windows
  - Handles input events from kernel
  - Composites final screen display
  - Communicates with GPU for rendering

- **code-first_server**: Wayland Server on the backend
  - Runs framework application logic
  - Operates headless browser for rendering
  - Provides shared video memory buffers
  - Manages buffer lifecycle

**magentic_mesh**
- **Browser WebSocket Client**: Client-side semantic communication
  - Encodes/decodes JSON-RPC-LD messages
  - Manages WebSocket connections
  - Ensures semantic clarity in requests

- **Server WebSocket Handler**: Server-side AI integration
  - Parses JSON-RPC-LD messages
  - Validates against SHACL shapes
  - Manages LLM context
  - Interfaces with LLM services

**Legacy Framework Support**
- Rails (Ruby)
- Laravel (PHP)
- Express (TypeScript)
- Flask (Python)
- Django (Python)

### Key Relationships
- Wayland Protocol enables efficient buffer sharing between browser and server
- JSON-RPC-LD provides semantic communication over WebSockets
- Multiple language implementations support diverse ecosystems
- Framework adapters integrate existing applications

---

## 2. Class Diagram

### Purpose
Provides detailed view of the core classes in CodeFirst, their attributes, methods, and relationships.

### Browser-Side Classes

**WaylandCompositor**
- Manages the scenegraph of application windows
- Processes input events from kernel
- Transforms coordinates between screen and window space
- Triggers recomposition and page flips
- Core responsibility: Compose final display from multiple buffers

**SceneGraph**
- Maintains hierarchical structure of windows
- Finds target window for input events
- Applies coordinate transformations
- Updates node transformations

**ApplicationWindow**
- Represents individual application window
- Holds reference to shared buffer
- Manages position and transformations
- Handles input events

**WebSocketClient**
- Manages WebSocket connections to servers
- Uses JsonRpcLdEncoder for outgoing messages
- Uses JsonRpcLdDecoder for incoming messages
- Provides Promise-based API for requests

**JsonRpcLdEncoder/Decoder**
- Builds JSON-RPC-LD messages with @context
- Validates message structure
- Extracts semantic information
- Resolves IRIs from contexts

### Server-Side Classes

**WaylandServer**
- Creates and manages shared buffers
- Notifies compositor of damage regions
- Coordinates with compositor for buffer lifecycle
- Provides buffers to headless browser

**HeadlessBrowser**
- Renders HTML/CSS content
- Outputs to shared video memory buffers
- Supports screenshot capture
- Manages rendering pipeline

**WebSocketHandler**
- Routes incoming JSON-RPC-LD requests
- Validates messages with SHACL
- Delegates to framework adapters
- Returns semantically annotated responses

**LlmManager**
- Analyzes user intent using LLM services
- Generates contextual responses
- Updates conversation context
- Interfaces with external LLM APIs

**ContextManager**
- Maintains conversation contexts per session
- Stores interaction history
- Provides context for LLM requests
- Manages context lifecycle

**FrameworkAdapter (Interface)**
- Defines integration contract for frameworks
- Implemented by RailsAdapter, DjangoAdapter, ExpressAdapter
- Handles framework-specific initialization
- Routes requests to framework routers

### Protocol Classes

**JsonRpcLdMessage**
- Represents a JSON-RPC-LD message
- Contains method, params, context, and id
- Mandatory @context for semantic meaning

**JsonLdContext**
- Maps terms to IRIs
- Resolves semantic meaning
- Enables machine understanding

**ShaclShape**
- Defines validation rules
- Specifies property constraints
- Validates data structures
- Ensures type safety

**ValidationResult**
- Reports validation outcome
- Contains error details
- Indicates conformance

---

## 3. Deployment Diagram

### Purpose
Shows how CodeFirst components are deployed across client machines, application servers, and external services.

### Deployment Nodes

**Client Machine**
- Runs standard web browser
- Hosts code-first_browser with Wayland Compositor
- Connects to multiple application servers
- Receives input from kernel (evdev)
- Controls display via KMS

**Application Servers (Multiple)**
- Each can run different language/framework combination
  - Server 1: Rails (Ruby)
  - Server 2: Django (Python)
  - Server 3: Express (TypeScript)
- Each contains:
  - Framework application
  - Headless browser
  - Wayland server
  - JSON-RPC-LD handler
- Can scale independently

**External Services**
- **LLM Services**: GPT-4, Claude, custom models
  - Accessed via HTTPS/API
  - Provides intent analysis and generation
- **Shared Database**: PostgreSQL, MySQL, MongoDB
  - Accessed via SQL/NoSQL protocols
  - Can be shared or isolated per service

**Linux Kernel**
- Provides KMS (Kernel Mode Setting) for display
- Provides evdev for input devices
- Interfaces with browser compositor

### Communication Protocols
- **Wayland Protocol**: Binary protocol for buffer sharing
- **WebSocket + JSON-RPC-LD**: Semantic communication
- **HTTPS/API**: External service access
- **SQL/NoSQL**: Database access

---

## 4. Use Case Diagram

### Purpose
Illustrates the various use cases supported by CodeFirst from the perspective of different actors.

### Actors

**End User**
- Views application UI
- Inputs data
- Navigates interface
- Receives visual feedback
- Executes commands

**AI Agent**
- Analyzes user intent
- Processes natural language
- Generates responses
- Maintains context
- Provides recommendations

**Developer**
- Integrates legacy frameworks
- Implements business logic
- Defines SHACL shapes
- Configures Wayland server
- Deploys applications

**System Administrator**
- Monitors performance
- Manages LLM services
- Configures security
- Scales services

### Key Use Case Relationships

**User → Agent Interactions**
- User commands trigger intent analysis
- Agent responses result in visual feedback
- Continuous context preservation

**Developer → System**
- Framework integration includes SHACL definition
- Deployment includes Wayland configuration
- Business logic uses data management capabilities

**Communication Layer**
- All semantic messages use JSON-RPC-LD
- All messages validated with SHACL
- Visual updates use Wayland buffer sharing

---

## 5. Sequence Diagram: User Input Event Handling

### Purpose
Demonstrates how user input events flow through the application_mesh from kernel to display update.

### Flow Steps

1. **User Interaction**: User performs mouse click or keyboard input
2. **Kernel Processing**: Kernel captures event via evdev driver
3. **Event Delivery**: Kernel sends event to Wayland Compositor
4. **Scenegraph Query**: Compositor queries scenegraph to find target window
5. **Coordinate Transformation**: Compositor applies inverse transformations (screen → window coordinates)
6. **Event Routing**: Compositor sends event to appropriate Wayland Server via Wayland protocol
7. **Application Processing**: Server's framework application processes event and updates state
8. **Rendering**: Application renders updated UI in headless browser
9. **Buffer Update**: Headless browser renders to shared video memory buffer
10. **Damage Notification**: Server notifies compositor of changed regions
11. **Recomposition**: Compositor recomposites screen using updated buffers
12. **Display Update**: Compositor schedules pageflip with GPU (KMS)
13. **Visual Feedback**: User sees updated UI

### Key Advantages
- **Direct Path**: No X server middleman
- **Efficient**: Shared buffers avoid copying
- **Accurate**: Compositor understands all transformations
- **Fast**: Direct GPU access for pageflips

---

## 6. Sequence Diagram: LLM Interaction

### Purpose
Shows how AI agent interactions work through the magentic_mesh using JSON-RPC-LD for semantic communication.

### Flow Steps

1. **User Input**: User enters natural language query ("Create sales dashboard")
2. **Request Building**: Browser builds JSON-RPC-LD request with @context
3. **Message Encoding**: Encoder creates semantically annotated message
4. **WebSocket Transmission**: Message sent over WebSocket connection
5. **Parsing**: Server parses JSON-RPC-LD message
6. **SHACL Validation**: Validator checks structure and types against shapes
7. **Context Retrieval**: Server retrieves conversation context
8. **LLM Analysis**: External LLM service analyzes intent with context
9. **Action Execution**: Application logic executes required actions (query DB, generate charts)
10. **Context Update**: Interaction stored in context manager
11. **Response Building**: Server creates JSON-RPC-LD response with @context
12. **Response Validation**: Response validated against SHACL shapes
13. **WebSocket Return**: Response sent back over WebSocket
14. **Decoding**: Browser decodes semantic response
15. **UI Update**: Browser displays results to user

### Key Features
- **Semantic Clarity**: Mandatory @context prevents ambiguity
- **Type Safety**: SHACL validation ensures data integrity
- **Context Preservation**: Conversation history maintained
- **Machine Readable**: All data has explicit semantic meaning

---

## 7. Sequence Diagram: Complete Magnetic Application Flow

### Purpose
Illustrates the complete end-to-end flow of a magnetic (man/agent) application, showing how both mesh systems work together.

### Six Phases

**Phase 1: User Intent Capture**
- User clicks "Create Report" button
- Wayland Compositor processes input event
- WebSocket Client triggered to send request

**Phase 2: Semantic Processing**
- JSON-RPC-LD request validated with SHACL
- LLM Manager analyzes user intent
- External LLM service provides intent analysis
- Parsed intent: "Generate sales report for Q4"

**Phase 3: Application Logic Execution**
- Application queries database for sales data
- Data processed and metrics calculated
- Headless browser renders report UI
- HTML/CSS generated with charts
- Content rendered to shared buffer

**Phase 4: Visual Composition**
- Wayland Server updates buffer
- Damage notification sent to compositor
- Compositor recomposites screen with new buffer

**Phase 5: Response Communication**
- Compositor acknowledges update complete
- Server builds JSON-RPC-LD response with metadata
- Response sent to browser
- Browser updates UI state

**Phase 6: Continuous Interaction**
- User interacts with report (filter, drill-down)
- Cycle repeats with preserved context
- Seamless integration of human and agent

### Magnetic Characteristics

**Man (Human)**
- Natural interaction via UI
- Visual feedback via Wayland
- Intuitive workflows

**Agent (AI)**
- Semantic understanding via JSON-RPC-LD
- Context-aware responses
- Intelligent assistance

**Magnetic (Seamless Integration)**
- Unified interface
- Preserved context
- Efficient communication
- Real-time collaboration

---

## Technical Benefits

### Performance
- **Reduced Latency**: Direct compositor-to-hardware path
- **Efficient Memory**: Shared video buffers avoid copying
- **Parallel Processing**: Multiple servers render simultaneously

### Reliability
- **Type Safety**: SHACL validation prevents errors
- **Clear Contracts**: Machine-readable API documentation
- **Semantic Clarity**: Explicit meaning prevents misunderstandings

### Maintainability
- **Loose Coupling**: Semantic protocols enable independent evolution
- **Framework Flexibility**: Multiple languages and frameworks supported
- **Legacy Integration**: Existing applications easily wrapped

### Scalability
- **Distributed Composition**: Multiple servers provide buffers to single compositor
- **Language Diversity**: Different services use optimal languages
- **Incremental Adoption**: Gradual migration path

---

## Conclusion

These UML diagrams provide a comprehensive view of the CodeFirst architecture, demonstrating how Wayland and JSON-RPC-LD work together to enable magnetic (man/agent) applications. The combination of efficient visual rendering through Wayland's direct buffer sharing and semantic communication through JSON-RPC-LD's linked data approach creates a powerful platform for building applications where humans and AI agents collaborate seamlessly.

The architecture's support for multiple programming languages and legacy frameworks, combined with its emphasis on type safety and semantic clarity, makes CodeFirst an ideal choice for organizations looking to build next-generation collaborative applications.
