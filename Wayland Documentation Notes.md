# Wayland Documentation Notes

## Overview
Wayland is a replacement for the X11 window system protocol and architecture with the aim to be easier to develop, extend, and maintain.

## Key Concepts

### What is Wayland?
Wayland serves multiple purposes in the display server ecosystem:

1. **Protocol (Language)**: Wayland is the language (protocol) that applications use to communicate with a display server to make themselves visible and receive user input.

2. **System Architecture**: Beyond just a protocol, Wayland represents a complete system architecture. Unlike X11 which has a single common server (Xorg), Wayland has many compositor implementations, with each graphical environment bringing its own compositor.

### Core Components

#### Wayland Compositor
- A Wayland server is called a **"compositor"**
- Applications are Wayland **clients**
- There is no single common Wayland server like Xorg for X11
- Every graphical environment brings one of many compositor implementations
- Window management and end user experience are often tied to the compositor rather than being swappable components

#### libwayland
A core part of Wayland architecture is **libwayland**, which is:
- An inter-process communication (IPC) library
- Translates protocol definitions in XML to C language API
- Does **not** implement Wayland itself
- Merely encodes and decodes Wayland messages
- Actual implementations are in various compositor and application toolkit projects

### Flexibility and Use Cases
Wayland does not restrict where and how it is used. A Wayland compositor can be:

1. **Standalone Display Server**: Running on Linux kernel modesetting and evdev input devices or on many other operating systems

2. **Nested Compositor**: Itself an X11 or Wayland application (client)

3. **Application-Internal Communication**: Used within applications, as done in some web browsers

## Relevance to CodeFirst

### In application_mesh
In CodeFirst's **application_mesh**, Wayland plays a critical role:

- **code-first_browser** acts as a **Wayland Compositor**
  - Browser-side of CodeFirst application
  - Can manage multiple application windows
  
- **code-first_server** includes a **Wayland Server**
  - Server-side of CodeFirst application
  - Renders application-specific pages in headless browser
  - Communicates rendered content via Wayland protocol to application windows in the compositor

This architecture allows the browser to compose UX from multiple application endpoints, with Wayland serving as the communication protocol between the server-rendered content and the client-side compositor.

## Additional Resources
- Wayland architecture documentation
- Wayland FAQ
- Full documentation available at wayland.freedesktop.org

## Wayland Architecture Deep Dive

### Understanding the Difference from X11

The best way to understand Wayland architecture is to follow an event from input device to screen display.

#### X11 Architecture (Traditional)
The X11 system has several problems:

1. **Kernel** → **X Server** (via evdev): Kernel sends input events to X server
2. **X Server** → **Clients**: X server determines which window receives the event (but doesn't know transformations applied by compositor)
3. **Client** → **X Server**: Client sends rendering request back
4. **X Server** → **Compositor**: X server calculates damage region and notifies compositor
5. **Compositor** → **X Server**: Compositor renders through X server
6. **X Server** → **Hardware**: X server handles buffer swaps/pageflips

**Problems with X11 approach:**
- X server doesn't have information to properly decide which window receives events
- Cannot transform screen coordinates to window-local coordinates correctly
- X server acts as unnecessary middleman between applications and compositor
- Extra context switches between compositor and hardware
- Most complexity X server handled is now in kernel or self-contained libraries (KMS, evdev, mesa, etc.)

#### Wayland Architecture (Modern)
**Key principle: The compositor IS the display server**

Control of KMS (Kernel Mode Setting) and evdev transferred to compositor. Direct communication paths:

1. **Kernel** → **Compositor**: Kernel sends input events directly to compositor
2. **Compositor** → **Client**: Compositor uses scenegraph to determine target window and transforms coordinates
3. **Client** → **Compositor**: Client renders locally and sends damage region notification
4. **Compositor** → **Hardware**: Compositor directly issues ioctl to schedule pageflip with KMS

**Advantages:**
- Compositor understands all transformations applied to windows
- Can compute inverse transformations for input events
- Direct path from compositor to hardware
- Eliminates unnecessary context switches
- Compositor has complete control over scenegraph

### Wayland Rendering Model

#### Direct Rendering
With Wayland, rendering uses **direct rendering** where client and compositor share video memory buffers.

**Process:**
1. Client links to rendering library (e.g., OpenGL)
2. Client renders directly into shared buffer
3. Compositor uses buffer as texture when compositing desktop
4. Client only needs to tell compositor which buffer to use and where content changed

#### Two Buffer Update Strategies

**Strategy 1: Buffer Swapping**
- Render new content into new buffer
- Tell compositor to use new buffer instead of old
- Application can allocate new buffer each time or cycle between multiple buffers
- Buffer management entirely under application control

**Strategy 2: Back Buffer Rendering**
- Render into back buffer first
- Copy to compositor surface
- Avoids race conditions with compositor
- Prevents flickering from partial renders
- Back buffer can be allocated on-fly or kept around

**Important:** Application must always tell compositor which area holds new content, even when swapping buffers (only small part may have changed, like cursor or spinner).

### Hardware Enabling for Wayland

#### Requirements
- Modesetting/display support
- EGL/GLES2 support
- Efficient buffer sharing between processes

#### Client Side: Wayland EGL Platform
Defines native types and creation methods:
- EGLNativeDisplayType
- EGLNativeWindowType
- EGLNativePixmapType

**Implementation:**
- Full API in `wayland-egl.h` header
- Open source implementation in mesa EGL stack: `platform_wayland.c`
- Vendor-specific protocol extension for buffer detail communication

### CodeFirst Integration

In CodeFirst's **application_mesh**:

**code-first_server (Wayland Server)**
- Renders application pages in headless browser
- Acts as Wayland server providing buffers
- Communicates rendered content via Wayland protocol

**code-first_browser (Wayland Compositor)**
- Acts as compositor receiving buffers from server
- Manages multiple application windows
- Composes final UX from multiple application endpoints
- Handles input events and coordinates transformations

This architecture allows CodeFirst to leverage Wayland's efficient buffer sharing and direct rendering capabilities for seamless integration of server-rendered content into browser-based composite interfaces.
