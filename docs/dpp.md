# INTERFACER Integration - ValueFlows Modelling
## Value Flows API description
- https://www.valueflo.ws/specification/spec-overview/
- https://www.valueflo.ws/specification/json-schemas/EconomicEvent.json 

### Concepts
- Environmental Accounting: https://www.valueflo.ws/concepts/ecology/#agents
    - TODO check links to CO2 emissions
- Production Examples: https://www.valueflo.ws/examples/ex-production/#manufacturing
    - First 3D Printing Service Description
- Analysis: https://www.valueflo.ws/concepts/estimates/#connecting-plans-and-scenarios
    - CHECK for LAUDS use-case for comparison of different machining processes regarding energy and material usage, other parameters?
    - CHECK connection with LAUDS gateway data sets from machining
        - https://www.valueflo.ws/specification/model-text/#flows-in-motion-recipe
            - CHECK node-red implementation possibilities



## LAUDS Use-Case Modelling 3d print example

**Use-Case: Machining Service**

- (first draft, reference example -> https://github.com/interfacerproject/Interfacer-notebook/blob/main/gownshirt_flow.1.1.txt)
- first draf, jupyter notebook/ IF-API implementation -> https://github.com/new-production-institute/lauds-gateway-jupyter-notebook/tree/main/IF-integration
- translate LAUDS 3D printing machining use-case description in notebook using the gowns use-case as reference
----
```

A: Machine-Operator-"Adam"
RS: Machine-3D-Printer
RS: Filament
RS: Electrical-Energy

E: raise Machine-3D-Printer (1, Machine-Operator-"Adam")
R: Machine-3D-Printer

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

P: create 3D-object-"Luffy"

E: produce 3D-object-"Luffy" (1, Machine-
Operator-"Adam")

R: 3D-object-"Luffy"

```
- reference notebook for jupyter notebook
    - https://github.com/new-production-institute/lauds-gateway-jupyter-notebook/tree/main/interfacer_machining_model_3d-printer
- Grafical representation with additional flows for recycling and Gcode file creation as software engineering activities

![](https://pad.fabcity.hamburg/uploads/edb55caf-f74c-4572-9c4d-ac8008980c1b.png)
![](https://pad.fabcity.hamburg/uploads/74958891-4ae6-401a-917a-c9ac9d826b08.png)
