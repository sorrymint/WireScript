# WireScript Language Documentation (Proposed)

[//]: # (v0.1)

This document outlines the proposed rules and features for the WireScript Domain-Specific Language (DSL), focusing on
its syntax, structure, and application in designing residential electrical systems.

## 1. General Syntax & Structure

WireScript is a block-structured language designed for readability and simplicity.

- Blocks: The core building blocks of the language. They are defined by a keyword and a label, followed by a set of
  properties and nested blocks enclosed in curly braces `{}`.
- Properties: Key-value pairs that define the characteristics of a block. Each property ends with a semicolon `;`.
- Comments: Single-line comments start with `//`. Multi-line comments are enclosed in `/* ... */`.

**Example:**

```wirescript
// This is a single-line comment

/*
    This is a multi-line comment.
    It can span multiple lines.
*/

// A panel block with a label and properties
panel "Garage Subpanel" {
    amperage: 100A;
    slots: 20;
}
```

## 2. Keywords and Core Concepts

The language is built around a set of keywords that represent real-world electrical components and concepts.

- **service**: The point of entry for electrical power to the building.
- **panel**: A distribution panel (main or sub) that houses breakers.
- **area**: A defined physical space within a structure, like a room or a garage.
- **circuit**: A specific electrical path originating from a breaker.
- **breaker**: A safety device in a panel that protects a circuit.
- **run**: The wire that forms the physical path of a circuit.
- **outlet, fixture, appliance**: Devices that consume power.
- **switch**: A control device that opens or closes a circuit.
- **series, parallel**: Blocks that define the configuration of a group of devices.

## 3. Data Types and Values

WireScript uses simple, intuitive data types for defining properties.

- **Strings**: Enclosed in double quotes (`"..."`). Used for labels and text values.
- **Numbers**: Integers or decimals, often followed by a unit (e.g., `A` for Amps, `V` for Volts, `ft` for feet).
- **Booleans**: The keywords `true` or `false`.
- **Labels**: A special type of string used to uniquely identify blocks, allowing for connections between them.

**Example:**

```wirescript
service {
    voltage: 240V;     // Number with unit
    type: "residential"; // String
    amperage: 200A;    // Number with unit
}

outlet "Bedroom 1", gfci: true; // String and Boolean
```

## 4. Connectivity and Logic

WireScript defines connections and logical groupings using specific properties and nested blocks.

- **breaker: "..."**: Establishes a connection between a circuit and a named breaker in a panel or subpanel.
- **controls: "..."**: Links a switch to a named fixture or a group of devices.
- **series and parallel blocks**: Explicitly define how devices within a circuit are wired relative to each other.
  `parallel` is the default behavior.

## 5. Hierarchy and Attachment Rules

The language's structure mirrors the physical hierarchy of an electrical system.

- A `service` block is the top-level entry point.
- `panel` and `area` blocks are defined at the root level.
- A `panel` block can represent a main panel or a subpanel. A subpanel is defined within the area block it serves and
  includes a `source` attribute that points to the label of the breaker that powers it.
- An `area` block represents a physical space and uses a `source` attribute to identify the panel or subpanel that
  provides power to all its circuits.
- `breaker` blocks must be defined within a `panel`.
- `circuit` blocks must be defined within an `area`.
- Devices (`outlet`, `fixture`, `appliance`, `switch`) must be defined within a `circuit` or a `series`/`parallel`
  block.

### Examples of Hierarchy and Connectivity:

To connect everything, you define a service at the top level. The main panel is also at the top level and represents the
primary distribution point. A panel representing a subpanel is defined within an area, and its source property is a
string that matches the label of a breaker in the main panel. Each area block must have a source property that links it
to a panel or subpanel label, making the power flow explicit. Finally, each circuit within an area links to a specific
breaker label from that area's source panel.

```wirescript
// Define the entry point for power
service "Residential Service" {
    voltage: 240V;
    type: "residential";
    amperage: 200A;
}

// Define the main panel at the top level
panel "Main Panel" {
    amperage: 200A;
    slots: 40;
    // The main panel has a breaker that feeds the garage subpanel
    breaker 60A, double_pole, label: "Garage Subpanel Feed";
    breaker 20A, label: "Outdoor Outlets";
}

// The garage subpanel is defined at the top level, linking to its source breaker
panel "Garage Subpanel", source: "Garage Subpanel Feed" {
    amperage: 125A;
    slots: 20;
    breaker 15A, label: "Workshop Lights";
}

// The garage area is powered by the main panel
area "Garage" {
    // A circuit in the garage linking to a breaker in the Main Panel
    circuit "Outdoor Outlets" {
        breaker: "Outdoor Outlets";
        run #12-2, length: 50ft;
        outlet "Patio", gfci: true;
    }
}

// The workshop area is powered by the garage subpanel
area "Workshop" {
    // A circuit in the workshop linking to a breaker in the Garage Subpanel
    circuit "Workshop Lights" {
        breaker: "Workshop Lights";
        run #14-2, length: 30ft;
        fixture "Work Light 1", load: 0.5A;
    }
}
```

## 6. Load Configurations

WireScript supports two primary load configurations for electrical devices: series and parallel. All devices are in a
parallel configuration by default unless otherwise specified.

### 6.1 Parallel Configuration

In a parallel circuit, all devices receive the full circuit voltage. The current drawn by each device is additive, and
the total circuit amperage is the sum of all individual device loads. This is the standard configuration for most
residential wiring.

**Definition:**

```wirescript
// A parallel circuit is the default
circuit "Parallel Outlets" {
    // The breaker must be defined in the parent area's source panel
    breaker: "Garage Outlets";
    // The wiring run
    run #14-2;
    // Devices in parallel
    outlet "Outlet 1", load: 0.5A;
    outlet "Outlet 2", load: 0.5A;
}
```

**Example:**
A standard circuit with multiple outlets is a good example of a parallel configuration. Each outlet receives 120V, and
the total load on the circuit is the sum of the loads of the devices plugged into the outlets.

### 6.2 Series Configuration

In a series circuit, the components are arranged end-to-end, so the current has only one path to flow. The voltage is
divided among the devices, and the total resistance is additive. This configuration is rare in residential power wiring
but common in low-voltage electronics (e.g., decorative light strings).

**Definition:**

```wirescript
// A series circuit must be explicitly defined
circuit "Series Lights" {
    breaker: "Hallway Lights";
    run #18-2;
    // Devices are explicitly defined in a series block
    series {
        light "LED 1", voltage: 3V, load: 0.1A;
        light "LED 2", voltage: 3V, load: 0.1A;
        light "LED 3", voltage: 3V, load: 0.1A;
    }
}
```

**Example:**
An old string of Christmas lights where if one bulb burns out, the entire string goes dark, is a classic example of a
series circuit.

## 7. 240-Volt Split-Phase System

The language recognizes and validates the specific configuration of the US residential split-phase electrical system,
where 240V is derived from two 120V lines that are 180 degrees out of phase.

### 7.1 Split-Phase Definition

WireScript handles this by defining a `double_pole` breaker and associating it with devices that require 240V. This
tells the parser that the circuit uses both hot legs of the incoming service.

**Definition:**

- **double_pole**: A property on a breaker to indicate it connects to two hot legs.
- **voltage**: A property on an appliance or fixture to specify its operating voltage. A value of 240V triggers the need
  for a double_pole breaker.

**Example:**

```wirescript
panel "Main Panel" {
    // A 30A double-pole breaker
    breaker 30A, double_pole, label: "Water Heater";
}

area "Utility Room", source: "Main Panel" {
    circuit "Water Heater" {
        // Connects to the double-pole breaker in the Main Panel
        breaker: "Water Heater";
        // Requires a 2-hot, 1-ground wire (#10-3)
        run #10-3, length: 20ft;
        // Appliance is explicitly defined as 240V
        appliance "Electric Water Heater", voltage: 240V, load: 20A;
    }
}
```

## 8. Keyword and Attribute Outline

Here is a full breakdown of every keyword and its valid attributes.

### service

- **voltage**: Number with unit V. The supply voltage.
- **type**: String. residential, commercial, etc.
- **amperage**: Number with unit A. The total available amperage.

### panel

- **amperage**: Number with unit A. The panel's rating.
- **slots**: Number. The number of available breaker slots.
- **source**: String. The label of the breaker that feeds this panel (if applicable).

### area

- **source**: String. The label of the panel or subpanel that powers this area.
- No other attributes. Used as a container with a label.

### breaker

- **amperage**: Number with unit A. The breaker's rating.
- **double_pole**: Boolean (true or false). Indicates a 240V breaker.
- **label**: String. A unique identifier for the breaker.

### circuit

- **breaker**: String. The label of the breaker that powers this circuit.
- **run**: Nested run block.

### run

- **wire_gauge_and_conductors**: String (e.g., #14-2, #10-3, #8-4).
- **length**: Number with unit ft.

### outlet

- **label**: String. Unique identifier.
- **load**: Number with unit A. The device's current draw.
- **gfci**: Boolean (true or false). Indicates a Ground-Fault Circuit Interrupter.
- **afci**: Boolean (true or false). Indicates an Arc-Fault Circuit Interrupter.

### appliance

- **label**: String. Unique identifier.
- **voltage**: Number with unit V.
- **load**: Number with unit A.

### fixture

- **label**: String. Unique identifier.
- **type**: String (e.g., recessed, pendant, chandelier).
- **voltage**: Number with unit V.
- **load**: Number with unit A.
- **count**: Number. The number of identical fixtures.

### switch

- **label**: String. Unique identifier.
- **type**: String. 3-way, 4-way, etc.
- **controls**: String. The label of the device or group it controls.

### series

- No attributes. A container block for devices.

### parallel

- No attributes. A container block for devices.

## 9. Comprehensive Example

This example demonstrates how all the rules and features can be used together to define a complete kitchen electrical
system.

```wirescript
service {
    voltage: 240V;
    type: "residential";
    amperage: 200A;
}

panel "Main Panel" {
    amperage: 200A;
    slots: 40;

    breaker 20A, label: "Kitchen Outlets";
    breaker 15A, label: "Kitchen Lights";
    breaker 60A, double_pole, label: "Garage Subpanel Feed";
    breaker 40A, double_pole, label: "Electric Range";
}

panel "Garage Subpanel", source: "Garage Subpanel Feed" {
    amperage: 100A;
    slots: 20;

    breaker 20A, label: "Garage Outlets";
}

area "Kitchen" {
    // General purpose outlet circuit, powered from the Main Panel
    circuit "Countertop Outlets" {
        breaker: "Kitchen Outlets";
        run #12-2, length: 25ft;
        outlet "Countertop 1", gfci: true;
        outlet "Countertop 2", load: 1.5A;
    }

    // A 240V appliance circuit, powered from the Main Panel
    circuit "Electric Range" {
        breaker: "Electric Range";
        run #8-4, length: 10ft;
        appliance "Electric Range", voltage: 240V, load: 30A;
    }

    // A lighting circuit
    circuit "Kitchen Lights" {
        breaker: "Kitchen Lights";
        run #14-2, length: 15ft;
        fixture "Recessed Lights", count: 4, load: 0.8A;
        fixture "Pendant Light", load: 0.5A;
    }
}

area "Garage" {
    // Garage outlets powered from the Garage Subpanel
    circuit "Garage Outlets" {
        breaker: "Garage Outlets";
        run #12-2, length: 50ft;
        outlet "Garage Outlet 1", gfci: true;
        outlet "Garage Outlet 2", load: 1A;
    }
}
```
