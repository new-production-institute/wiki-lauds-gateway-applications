# Digital Product Passport generation

To generate a Digital Product Passport (DPP) the [INTERFACER notebook](https://github.com/interfacerproject/Interfacer-notebook) provided by [Dyne.org](https://dyne.org/) is integrated in the LAUDS Gateway stack.

## Basic concept with ValueFlows
- Specification: https://www.valueflo.ws/specification/spec-overview/
- Economic modelling: https://www.valueflo.ws/specification/json-schemas/EconomicEvent.json 
- Environmental Accounting: https://www.valueflo.ws/concepts/ecology/#agents
- Production Examples: https://www.valueflo.ws/examples/ex-production/#manufacturing
- Analysis: https://www.valueflo.ws/concepts/estimates/#connecting-plans-and-scenarios

## DPP fo LAUDS Manufacturing Use-Case: 3D printing service as a process

**Use-Case description**

A machine operator uses a 3D printer to produce a 3D object, consuming filament and electrical energy and citing a Gcode file.

**Use-Case definition**

```

A: Machine-Operator-"Adam" // agent
RS: Machine-3D-Printer // resource specification
RS: Filament
RS: Electrical-Energy

E: raise Machine-3D-Printer (1, Machine-Operator-"Adam") //event
R: Machine-3D-Printer // resource

E: raise Filament (1kg, Machine-Operator-"Adam")
R: Filament

E: raise Electrical-Energy (XXX Wh, Machine-
Operator-"Adam")

R: Electrical-Energy

E: consume Filament (XXX cm³, Machine-
Operator-"Adam")

E: consume Electrical-Energy (XXX Wh, Machine-
Operator-"Adam")

E: cite 3D-gcode-file"Luffy" (1, Machine-
Operator-"Adam")

E: use machining-3D-printing

P: create 3D-object-"Luffy" // process

E: produce 3D-object-"Luffy" (1, Machine-
Operator-"Adam")

R: 3D-object-"Luffy"

```

**Use-Case visual representation**

Grafical representation with optional additional flows for recycling process and Gcode file creation process to extend (as software engineering/ digital activity)

![](https://pad.fabcity.hamburg/uploads/8fcb4a66-38a6-44c8-a738-e1ab72a92d00.svg)

**Use-Case DPP trace renderd as a sankey diagram** 

![](https://pad.fabcity.hamburg/uploads/7ef8073b-356e-442e-82e0-3fc2fde7e460.png)


## 3D Printing Use Case Setup Notebook Documentation

This document provides guide to the `lauds_use_case_3dprinting_setup.ipynb` notebook. It explains each section's purpose, what it accomplishes, and where to customize parameters for your specific use case. The notebook sets up a decentralized 3D printing workflow using the Interfacer framework, integrating with Zenflows for tracking and DPP (Digital Product Passport) for traceability.

### Overview

The notebook initializes a 3D printing scenario involving users, locations, resources, and processes. It registers entities in the Zenflows system, defines specifications, and simulates production events. Customize variables like endpoints, user details, and resource amounts to fit your needs.

### 1. Module Imports and Auto-Reload Setup

**What it does:** Loads essential libraries for API interactions, data handling, and development convenience. Enables auto-reloading so changes to imported modules take effect without restarting the kernel.

### 2. Configuration and Endpoints

**What it does:** Defines core settings for the use case, including API URLs and participant lists. This establishes the connection to staging servers for Zenflows and DPP.

### 3. File Path Configuration

**What it does:** Generates file paths for storing data like user credentials, locations, and resources. Prints paths for verification, ensuring organized data persistence.

**Customization:** Paths are auto-generated from `ENDPOINT` and `USE_CASE`. No direct changes needed, but ensure the `use_cases/{USE_CASE}/` directory exists or is writable.

### 4. Initialize Data Structures

**What it does:** Sets up empty data containers and loads or creates sample user and location data. If files exist, it reuses them; otherwise, it generates defaults and saves to JSON.

**Where to insert parameters:**

- **User details:** Edit `users_data['A']` and `users_data['B']` with real values for `"userChallenges"` (security questions), `"name"`, `"username"`, `"email"`, and `"note"`.
- **Location details:** Modify `locs_data['A']` and `locs_data['B']` with `"name"`, `"lat"`, `"long"`, `"addr"`, and `"note"` for geographic info.

### 5. Authentication Setup: HMAC Generation

**What it does:** Retrieves or generates HMAC keys for secure user authentication in Zenflows.

**Customization:** Relies on prior user data and endpoints. No manual parameters—runs automatically for each user in `USERS`.

### 6. Cryptographic Key Generation

**What it does:** Loads or creates public-private key pairs for users, enabling encrypted transactions.

**Customization:** Uses user credentials file. No parameters to insert.

### 7. User Registration in Zenflows

**What it does:** Registers users in the Zenflows platform and assigns unique person IDs.

**Customization:** Depends on user data and endpoint. Automatic for listed users.

### 8. Location Registration and Assignment

**What it does:** Registers locations and links them to users for location-based tracking.

**Customization:** Uses location data from initialization. Note: There's a duplicate cell—run once.

### 9. Unit of Measurement Registration

**What it does:** Registers units like kilograms or watt-hours in the system for consistent resource quantification.

**Where to insert parameters:** In `get_unit_id` calls, specify:

- Label (e.g., `'mass'`), symbol (e.g., `'kg'`), and ontology (e.g., `'om2:kilogram'`).  
- Add or remove calls for custom units.

### 10. Process Definition

**What it does:** Creates a production process (e.g., 3D printing "Luffy") and saves it to file.

**Where to insert parameters:**

- `process_name`: Name of the process (e.g., `'Create_3D_object_Luffy'`).  
- `note`: Description including performer (auto-filled from user data).

### 11. Resource Specification Registration

**What it does:** Defines specs for resources like filament or the 3D printer, including classifications and units.

**Where to insert parameters:** For each resource spec in `get_resource_spec_id`:

- `name`: Resource name (e.g., `'Filament'`).  
- `note`: Description.  
- `classification`: Wikidata URL (e.g., `'https://www.wikidata.org/wiki/Q11782'`).  
- `default_unit_id`: From `units_data` (e.g., `units_data['mass']['id']`).

### 12. Initial Resource Creation

**What it does:** Instantiates starting resources (e.g., printer, filament) and saves them.

**Where to insert parameters:** In `get_resource` calls:

- `res_name`: Resource name (e.g., `'Filament'`).  
- `amount`: Quantity (e.g., 1 for kg; adjust for workflow).

### 13. Component Production

**What it does:** Simulates the 3D printing workflow by logging events: consuming inputs, citing files, and producing the output.

**Where to insert parameters:** For each step:

- `action`: Event type (e.g., `'consume'`, `'cite'`, `'produce'`).  
- `event_note`: Description (e.g., `'consume filament for 3D-object-Luffy'`).  
- `amount`: Quantity used/produced (e.g., 50 for grams).  
- For production, the new resource is auto-generated with a random ID.

### 14. Save All Component Data

**What it does:** Updates and saves the final resource data to file for use in production notebooks.