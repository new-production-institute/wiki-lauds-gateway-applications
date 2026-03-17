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

![](https://pad.fabcity.hamburg/uploads/74958891-4ae6-401a-917a-c9ac9d826b08.png)
