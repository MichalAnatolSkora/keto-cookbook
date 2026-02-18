# Keto Skills Documentation

This document defines core culinary skills as technical functions to ensure consistent, high-quality execution in a ketogenic environment.

## Skills

### 1. Searing Meat

**Name**: `skill_searing_meat`

**Description**:
Performs high-heat surface cooking of protein to trigger the Maillard reaction. This "function" is critical in keto cooking to generate depth of flavor (umami) without the use of sugar-based marinades or glazes. It creates a textural contrast between the crust and the interior.

**Input Schema**:
```json
{
  "surface_temp": {
    "type": "integer",
    "description": "Target temperature of the pan surface in Celsius.",
    "minimum": 200
  },
  "fat_type": {
    "type": "string",
    "enum": ["tallow", "ghee", "avocado_oil"],
    "description": "High smoke point fat compatible with keto macros."
  },
  "doneness": {
    "type": "string",
    "enum": ["rare", "medium-rare", "medium", "well-done"],
    "description": "Desired internal doneness of the protein."
  }
}
```

**Implementation Logic**:
1.  **Preheat**: Initialize pan to `surface_temp`. Verify temp using IR thermometer or water droplet test (Leidenfrost effect).
2.  **Dry**: Remove surface moisture from protein using paper towels. Moisture inhibits Maillard reaction.
3.  **Season**: Apply salt liberally to surface just before cooking.
4.  **Lubricate**: Add `fat_type` to pan. Wait until fat shimmers (viscosity change) or smokes slightly.
5.  **Execute**: Place protein in pan.
    *   *Constraint*: Do not overcrowd (steam generation error).
    *   *Action*: Press down verify even contact.
    *   *Wait*: Do not move protein until it releases naturally from the pan surface.
6.  **Flip**: Repeat for other side until internal temp matches `doneness` criteria.
7.  **Rest**: Transfer to neutral surface for 5-10 minutes to redistribute internal juices.

**Error Handling**:
*   `error: burning_fats`: If smoke is black/acrid, temperature > smoke point. **Action**: Discard fat, wipe pan, restart with lower temp or higher smoke point fat.
*   `error: gray_meat`: No crust formation. **Cause**: Pan too cold or meat too wet. **Action**: Remove meat, dry pat again, increase heat, retry.

---

### 2. Deglazing Pan

**Name**: `skill_deglazing_pan`

**Description**:
Retrieves caramelized proteins (fond) adhered to the pan after searing. This function solubilizes flavor compounds into a liquid medium, forming the base for keto-friendly pan sauces. Essential for resource efficiency and flavor maximization.

**Input Schema**:
```json
{
  "pan_state": {
    "type": "string",
    "description": "Condition of the pan post-searing.",
    "required_pattern": "contains_fond"
  },
  "liquid_agent": {
    "type": "string",
    "enum": ["bone_broth", "dry_wine", "vinegar", "water"],
    "description": "Solvent for the fond. Must be low-carb."
  },
  "volume_ml": {
    "type": "integer",
    "description": "Amount of liquid to add.",
    "minimum": 50
  }
}
```

**Implementation Logic**:
1.  **Assess**: Verify `pan_state`. Inspect fond color.
    *   *Check*: Golden/Dark Brown = OK. Black = `error: burned_fond`.
2.  **Heat Control**: Reduce heat to medium-high if pan is smoking excessively.
3.  **Execute**: Pour `liquid_agent` (`volume_ml`) into the hot pan.
    *   *Expectation*: Immediate vigorous boiling and steam release.
4.  **Scrape**: Use a wooden or silicone spatula to firmly scrape the bottom of the pan.
5.  **Dissolve**: Continue agitation until all solid fond particles are suspended in the liquid phase.
6.  **Reduce**: Simmer until liquid volume reduces by 50% to concentrate flavor.

**Error Handling**:
*   `error: burned_fond`: Fond is black and smells like charcoal. **Action**: Abort. Do not deglaze. The bitter taste is irretrievable. Clean pan and start sauce separately.
*   `error: weak_flavor`: Too much liquid or insufficient reduction. **Action**: Continue reduction process.

---

### 3. Emulsifying Sauce

**Name**: `skill_emulsifying_sauce`

**Description**:
Forces two immiscible liquids (fat and water/acid) into a stable colloidal suspension. In keto cuisine, this is the primary method for creating rich, creamy sauces (Hollandaise, Beurre Blanc, Mayo) without starch thickeners.

**Input Schema**:
```json
{
  "dispersed_phase": {
    "type": "string",
    "description": "The fat component (e.g., melted butter, oil)."
  },
  "continuous_phase": {
    "type": "string",
    "description": "The water/acid component (e.g., egg yolk, vinegar reduction)."
  },
  "method": {
    "type": "string",
    "enum": ["whisk", "blender", "immersion_blender"],
    "default": "whisk"
  },
  "temperature_c": {
    "type": "integer",
    "description": "Critical temperature for stability.",
    "maximum": 60
  }
}
```

**Implementation Logic**:
1.  **Base Setup**: Place `continuous_phase` in mixing vessel.
2.  **Initiate Agitation**: Begin `method` (whisking/blending) on the base.
3.  **Incorporate**: SLOWLY add `dispersed_phase`.
    *   *Rate*: Drop-by-drop initially, then typically a thin stream.
    *   *Logic*: `agitation_rate` must exceed `addition_rate` to break fat into microns.
4.  **Monitor**: Observe texture. It should thicken and become opaque.
5.  **Finalize**: Season once emulsion is stable.

**Error Handling**:
*   `error: broken_sauce`: Audit shows pools of oil separated from liquid. **Action (Recovery)**:
    1.  Get a clean bowl.
    2.  Add 1 tsp warm water (or fresh egg yolk).
    3.  Slowly whisk the *broken* sauce into the new base drop-by-drop.

---

### 4. Documenting Recipe

**Name**: `skill_documenting_recipe`

**Description**:
Standardized procedure for committing a new recipe to the repository. This function ensures that every culinary algorithm is accessible to both English and Polish speakers by enforcing bilingual input parameters.

**Input Schema**:
```json
{
  "title": {
    "en": "string (English Title)",
    "pl": "string (Polish Title)"
  },
  "description": {
    "en": "string (English Description)",
    "pl": "string (Polish Description)"
  },
  "ingredients": [
    {
      "item": "string",
      "quantity": "number",
      "unit": "string",
      "translation_pl": "string (Polish name of item and unit)"
    }
  ],
  "steps": [
    {
      "step_number": "integer",
      "instruction_en": "string",
      "instruction_pl": "string"
    }
  ],
  "macros_per_serving": {
    "net_carbs_g": "integer",
    "fat_g": "integer",
    "protein_g": "integer",
    "calories_kcal": "integer"
  }
}
```

**Implementation Logic**:
1.  **Initialize**: file `recipes/{category}/{recipe_name}.md`.
2.  **Validate**: Ensure all text fields have both `_en` and `_pl` values.
3.  **Format**: Render the recipe using the dual-language template.
4.  **Verify Macros**: accurate calculation of Net Carbs (< 10g/serving).

**Error Handling**:
*   `error: missing_translation`: If a field is provided in only one language. **Action**: Reject commit. Request translation.
*   `error: high_carb_alert`: If `net_carbs_g` > 20. **Action**: Mark as "Low Carb" instead of "Keto" or reject.

