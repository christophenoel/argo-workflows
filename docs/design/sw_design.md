# Software Design Document for [Project Name]

## Table of Contents
- [Software Components Design - Aspects of Each Component](#software-components-design---aspects-of-each-component)
    - [General](#general)
    - [Component Identifier](#component-identifier)
    - [Type](#type)
    - [Purpose](#purpose)
    - [Function](#function)
    - [Subordinates](#subordinates)
    - [Dependencies](#dependencies)
    - [Interfaces](#interfaces)
    - [Resources](#resources)
    - [References](#references)
    - [Data](#data)

## Software Components Design - Aspects of Each Component

### General
This section provides a detailed look at the design aspects of each component used in the software. It may be produced as part of a detailed design model if agreed upon with the customer.

### Component Identifier
- **Description**: Each component within the software is uniquely identified to maintain clear traceability and organization.
- **Naming Convention**: Components should be named according to the rules of the programming language or system, using a hierarchical naming scheme (e.g., `ParentName_ChildName`).

### Type
- **Logical Characteristics**: Describe the package, library, or class to which the component belongs.
- **Physical Characteristics**: Detail the component type using implementation terminology, such as task, subroutine, package, or file.

### Purpose
- **Description**: The purpose of each component is to explicitly link to the software requirements it implements, providing clear traceability from requirements through to implementation.

### Function
- **Overview**: Describe what the component does within the software architecture, using appropriate descriptions depending on the component type. Techniques like structured English, state-transition diagrams, or precondition-postcondition specifications can be employed.

> Note: Create a section of file for each function to detail the aspect 

### Subordinates
- **Description**: List the immediate children or subcomponents, describing how they interact or are utilized by the parent component. This includes any databases or files that are part of the component.

> Note: identify the DB, message broker, prometheus, etc.

### Dependencies
- **Description**: Detail any operations or conditions that must be met before this component can function properly, including exclusions during its operation.

> Note: if relevant, express dependency with other components (e.g. workflow needs module registry)

### Interfaces
- **Control Flow**: Describe how the component starts and terminates, including any interactions during execution (such as interrupts).
- **Data Flow**: Explain the input and output data flows, ensuring data structures are linked with control flows and interface components through common data areas or files.

Documentation detailing the API specifications and usage is provided in teh [API Design](./docs/design/api_design.md)

> NOte: sequence diagram and link to the api-design document.


### Resources
- **Requirements**: Itemize what the component needs from its environment to perform its function, excluding items that are part of the component interface.

> NOTE: include docker dependencies, might iclude maven dependencies (libs)


### Data
- **Internal Structures**: Describe the data structures internal to the component, including:
    - **Element Descriptions**: Name, type, dimension, and relationships between elements.
    - **Value Range**: Possible values for each element.
    - **Initial Values**: Start values for each element at initialization.

>Note: List tables/structure and Entities schemas but do not duplicate API data model (if provided in api doc)