# Plattar Architecture – Staggered Harmony Configurator

## Overview

This project uses Plattar to display a 3D model of the **Staggered Harmony tiny home**.  
The Plattar setup consists of a **Scene** containing two models:

1. Exterior Model
2. Interior Model

The configurator UI allows users to change **exterior cladding** options.  
These selections correspond to **model variations inside the Exterior model**.

The Interior model does not have variations and remains static.

---

# Plattar Structure

## Scene

The Scene is the top-level container that defines:

- Camera setup
- Lighting
- Environment
- Model placement
- Interaction context

The Scene contains the following models:

Scene
│
├── Exterior Model
└── Interior Model

The viewer loads the **Scene ID**, not individual models.

Example viewer initialization:

```html
<plattar-viewer scene-id="SCENE_ID"></plattar-viewer>



Models
Exterior Model

The Exterior model contains variations representing cladding types.

Each variation represents a full model variation in Plattar.

Available variations:

Ebony
Grey Friars
Sandstone
Corten

Selecting a cladding option in the configurator must trigger the corresponding Plattar variation ID.

Example runtime logic:

setVariation(exteriorModelInstanceId, variationId)
Interior Model

The Interior model exists in the scene but does not contain variations.

Interior customization is handled outside of Plattar using static images.

Therefore:

Interior model remains static

No variation switching required

Pricing Configuration System

The pricing system is defined in:

Pricing_STAGGERED_HARMONY.json

This file defines:

Categories
Options
Choices

Each choice contains:

id
label
price

Example:

{
  "id": "corrugated_iron_grey_friars",
  "label": "Corrugated Iron - Grey Friars",
  "price": 0
}
Plattar ID Registry

All Plattar IDs are stored in:

ID_REGISTRY_STAGGERED_HARMONY.json

This includes:

Scene IDs
Model instance IDs
Variation IDs

Example structure:

instanceId
instanceTitle
product
variations[]

Example variation entry:

{
  "id": "129b77b7-c712-45af-a0a3-311736dc8924",
  "title": "Grey Friars"
}
Required Mapping

The configurator must map:

Pricing Choice ID
→ Variation Name
→ Plattar Variation ID

Example mapping:

corrugated_iron_grey_friars
    ↓
Grey Friars
    ↓
129b77b7-c712-45af-a0a3-311736dc8924
Key Principle

The configurator must have one single source of truth that explicitly maps:

Pricing system
3D system
UI selections

This prevents ambiguity when switching model variations in Plattar.

The mapping file should therefore contain:

pricingChoiceId

pricingLabel

plattarVariationTitle

plattarVariationId

exteriorModelInstanceId

sceneId

This file will be used by the configurator logic to control the 3D viewer.