# Godot Repository Organization

A summary of the overall structure of the source code.

## Project Chunks And High-Level Overview

As a sequence of digestible chunks, the directories are organized like this:

- "Project":
    - `/`: Build config. C++ config. Git config.
    - `/tests`: Unit tests.
    - `/debug`: Logs.
    - `/doc`: XML API docs.
    - `/logs`: More logs.
    - `/misc`: Tons of random junk.
        - `/misc/ci`: Continuous Integration config
        - `/misc/dist`: Distribution-specific files. HTML wrappers. Some icons.
        - `/misc/hooks`: Git hooks. Automates script exec before/after git operations.
        - `/misc/scons`: Godot-specific scons dependencies.
        - `/misc/scripts`: Helper scripts for the Godot project.
- "Core":
    - `/thirdparty`: FOSS libraries that Godot uses.
        - Gives Godot access to work made by others.
    - `/drivers`: Godot-specific wrappers for `/thirdparty` and system stuff.
        - Gives Godot access to hardware on a platform.
    - `/core`: main data structures, libraries, and interfaces.
        - Gives Godot a foundation of common features for other sections.
        - Defines core OS interface.
    - `/platform`: Defines per-platform OS impls and entry points.
        - Scons builds engine for each platform based on these dir names.
        - Any shared platform code is in `/drivers` (unix OS).
    - `/servers`: Low-level, high-performance interfaces and impls, threaded.
        - Has rendering, physics, audio, window mgmt, nav, camera, XR.
    - `/scene`: High-level game framework using `/core` and `/servers`.
        - Has SceneTree, Node/Node2D/Spatial, etc. Configures low-level ops.
    - `/main`: Common platform ops. Setup. Start. Iteration. Cleanup.
        - Also handles cmdline args
- "Editor":
    - `/editor`: A C++-only Godot "game" with elevated privileges (tool mode).
        - Must only be dependent on "Core". No reference to `/modules`.
- "Modules":
    - `/modules`: Optional features appended to Godot Engine and/or Editor.
        - If a module needs Editor, wrap in `#ifdef TOOLS_ENABLED`.
        - If a module needs module, refactor to submodules (e.g. gdnative).

## 80% Accurate Dependency Tree

Dependencies between sections exist, but the code is similar to this:

- "Project":
    - `/`
- "Core":
    - `/main`
        - `/core`
        - `/scene`
        - `/platform`
            - `/drivers`
                - `/thirdparty`
        - `/servers`
            - `/drivers`
- "Editor":
    - `/editor`
        - "Core"
- "Modules":
    - `/modules`
        - "Core"
        - "Editor"
        - `/thirdparty`

## Directory Breakdown

A detailed summary of how each directory relates to the project.

### Source Code

- `/core`:
    - Description:
        - Basic data structures and algorithms.
        - Defines the core API for every Godot concept.
        - Low-level interfaces for use in other parts of Godot.
        - Variant
        - "Built-In" data types, e.g. Transform, String, Callable
        - Object, Reference, and Resource
        - ClassDB, [Property|Method|Signal]Info (Reflection system)
        - Scripting
        - Networking
        - IO
        - Core Engine Singletons
    - Dependencies:
        - `/server`: just RenderingServer for debugging
        - `/scene`: Nodes and Resources via forward declarations
    - Used By:
        Everything except `/thirdparty`.
- `/platform`:
    - Description:
        - Defines OS singleton implementations.
        - Defines entry points for each platform. Uses a common Main class.
        - Most OS-specific logic is handled here. Exception listed in Used By.
    - Dependencies:
        - `/core`: basic dependencies
        - `/drivers`: OS library dependencies
        - `/servers`: audio/rendering/window integrations
    - Used By:
        - `/servers`: concrete OS-specific DisplayServer impls
        - `/modules`: optional features: module interface, submodule impls
- `/drivers`:
    - Description:
        - Defines wrappers for local APIs that may be shared between platforms.
    - Dependencies:
        - `/core`: Basic dependencies
        - `/servers`: Vulkan uses DisplayServer info
        - extern local OS dependencies
    - Used By:
        - `/platform`: OS library dependencies
- `/editor`:
    - Description:
        - Defines the Godot Editor and its various subsystems.
        - Defines C++ EditorPlugin tools for anything in core, scene, etc.
    - Dependencies:
        - `/core`: Basic dependencies, UndoRedo
        - `/scene`: SceneTree, Node, Control, PackedScene, other nodes/resources
        - `/servers`: config, res mgmt, preview, window mgmt, debugging, settings
    - Used By:
        - `/modules`: EditorPlugins, protected by #ifdef TOOLS_ENABLED
- `/main`:
    - Description:
        - Defines the Main class. Boots up engine. Handles cmdline args.
    - Dependencies:
        - `/core`: Basic dependencies, type registration)
        - `/drivers`: type registration
        - `/modules`: type registration
        - `/platform`: type registration
        - `/scene`: SceneTree, Window, PackedScene, type registration
        - `/servers`: initialization, type registration
        - `/editor`: project manager, editor, docs generation
        - `/tests`: run unit tests upon request
    - Used By:
        None
- `/scene`:
    - Description:
        - Defines all objects/nodes/resources that are not required by /core.
        - Usage in other spaces is pretty self-explanatory.
    - Dependencies:
        - `/core`: Basic dependencies
        - `/servers`: access low-level APIs
    - Used By:
        - `/core`: Resource, Variant, other helpers with forward declarations
        - `/servers`: various Resources
        - `/editor`: tons of nodes/resources/etc.
        - `/modules`: tons of nodes/resources/etc.
- `/servers`:
    - Description:
        - Defines low-level Server interfaces.
        - Defines dependent Resources in some cases.
        - Defines concrete implementations of Server interfaces.
        - If multiple, either context-specific and auto-selected (2D/3D) or users are given the option to choose (e.g. Vulkan/GLES3).
    - Dependencies:
        - `/core`: basic dependencies
        - `/scene`: dependent resource types
    - Used By:
        - `/core`: grant friend access, window mgmt
        - `/drivers`: loosely used in wrapper code, e.g. DisplayServer::WindowID
        - `/editor`: config, res mgmt, preview, window mgmt, debugging, settings, etc.
        - `/main`: initialization, monitors, cmdline args config
        - `/scene`: usage in nodes/resources, etc.
        - `/modules`: usage in nodes/resources
            - Also platform-specific implementations for optional features
            - An increasing trend to use modules for implementations of Server
              interfaces?
                - Implementation of GdNavigationServer
                - Implementation of XRInterface for mobile and gdnative
- `/thirdparty`:
    - Description:
        Directly imports the source code of third-party libraries.
    - Dependencies:
        None
    - Used By:
        - `/modules`: integration
        - `/drivers`: integration
- `/modules`:
    - Description:
        Defines optional features which can easily be added or removed from
        both the engine and the editor during compilation.
    - Dependencies:
        Everything
    - Used By:
        None