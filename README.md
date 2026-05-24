# DrinkML v1.0
*Minimal viable specification — for Pound data model migration*
*Last Call Labs — May 2026*

---

## Overview

DrinkML is an open data interchange standard for beverages. This v1.0 document contains only what Pound needs to migrate its internal data model. Everything else is deferred to v2+.

---

## Record Structure

Every DrinkML record is a JSON object with these top-level fields:

```json
{
  "drinkml": "1.0.0",
  "id": "com.troegs:perpetual-ipa@2024",
  "name": "Perpetual IPA",
  "category": "beer",
  "abv": 0.075,
  "serving": {
    "size_oz": 16,
    "glass": "shaker_pint"
  },
  "color": {
    "hex": "#C47A1E",
    "opacity": 0.90
  },
  "ingredients": {
    "disclosure": "none"
  },
  "x-pound": {
    "icon_name": "beer-can",
    "sound_name": "pour_beer",
    "category_display": "Beer",
    "default_pour_oz": 16
  }
}
```

---

## Required Fields

| Field | Type | Description |
|---|---|---|
| `drinkml` | string | Spec version — always `"1.0.0"` for now |
| `id` | string | Unique identifier (see Namespace below) |
| `name` | string | Display name |
| `category` | enum | See Categories below |
| `abv` | float | Alcohol by volume as decimal (0.075 = 7.5%) |

---

## Optional Fields

| Field | Type | Description |
|---|---|---|
| `subtitle` | string | Short descriptor shown below name |
| `serving` | object | Recommended serving size and glass |
| `color` | object | Liquid color for draining glass visualization |
| `ingredients` | object | Ingredient list (see Ingredient Disclosure) |
| `nutrition` | object | Calories and carbs per serving |
| `tags` | array | Searchable tags |
| `x-pound` | object | Pound-specific display data |

---

## Namespace & ID Convention

DrinkML uses reverse domain notation. Domain ownership equals namespace ownership.

### Format
```
{reverse-domain}:{drink-slug}@{version-or-vintage}
```

### Examples
```
com.troegs:perpetual-ipa@2024
org.iba:negroni@classic
com.lastcalllabs:custom-negroni@v1
org.drinkml:light-lager@v1
```

### Reserved namespaces
| Namespace | Purpose |
|---|---|
| `org.drinkml` | Community-maintained canonical records |
| `org.iba` | IBA official cocktail list |
| `com.lastcalllabs` | Last Call Labs maintained records |

### For Pound's built-in database
All built-in Pound drink records use the `org.drinkml` namespace:
```
org.drinkml:light-lager@v1
org.drinkml:ipa@v1
org.drinkml:negroni@v1
org.drinkml:pinot-noir@v1
```

User-created custom drinks use `com.lastcalllabs:custom-{slug}@v1` until they publish to a proper namespace.

---

## Categories

| ID | Description |
|---|---|
| `beer` | Ales, lagers, stouts, porters, seltzers, hard ciders |
| `wine` | Still and sparkling wine |
| `cocktail` | Mixed drinks with defined recipes |
| `spirit` | Distilled beverages — neat, rocks, or simple mixer |
| `na` | Non-alcoholic and low-ABV (ABV < 0.005) |

---

## Serving Object

```json
{
  "serving": {
    "size_oz": 5,
    "glass": "bordeaux",
    "notes": "Standard restaurant pour"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `size_oz` | float | Recommended serving size in oz |
| `glass` | enum | Glass type (see Glass Types below) |
| `notes` | string | Optional serving notes |

### Glass Types

| ID | Name |
|---|---|
| `rocks` | Rocks / Old Fashioned |
| `highball` | Highball |
| `coupe` | Coupe |
| `martini` | Martini |
| `shaker_pint` | Shaker Pint |
| `tulip` | Tulip |
| `snifter` | Snifter |
| `flute` | Champagne Flute |
| `hurricane` | Hurricane |
| `shot` | Shot Glass |
| `burgundy` | Burgundy |
| `bordeaux` | Bordeaux |
| `mule` | Moscow Mule Mug |
| `nick_nora` | Nick & Nora |
| `beer_can` | Beer Can / Slim Can |
| `wine_glass` | Generic Wine Glass |

---

## Color Object

Used by Pound's draining glass visualization. Required for custom cocktails built in the cocktail builder — computed automatically from ingredients when available.

```json
{
  "color": {
    "hex": "#9B2335",
    "opacity": 0.95
  }
}
```

| Field | Type | Description |
|---|---|---|
| `hex` | string | RGB hex color of the liquid |
| `opacity` | float | 0.0 (clear) to 1.0 (opaque) |

---

## Ingredient Disclosure

Three tiers. Pound only needs ABV per ingredient for pace calculation — full recipes are optional.

### Disclosure tiers

| Value | What's shared |
|---|---|
| `none` | ABV only — no ingredients listed |
| `partial` | Some ingredients public, some marked proprietary |
| `full` | Complete ingredient list |

### None (ABV only)
```json
{
  "ingredients": {
    "disclosure": "none"
  }
}
```

### Partial
```json
{
  "ingredients": {
    "disclosure": "partial",
    "components": [
      {
        "name": "Gin",
        "brand": "Tanqueray",
        "abv": 0.473,
        "size_oz": 1.0,
        "published": true
      },
      {
        "name": "House Cordial",
        "abv": 0.05,
        "size_oz": 0.5,
        "published": false,
        "notes": "Proprietary"
      }
    ]
  }
}
```

### Full
```json
{
  "ingredients": {
    "disclosure": "full",
    "components": [
      {
        "name": "Gin",
        "brand": "Tanqueray",
        "abv": 0.473,
        "size_oz": 1.0
      },
      {
        "name": "Campari",
        "brand": "Campari",
        "abv": 0.25,
        "size_oz": 1.0
      },
      {
        "name": "Sweet Vermouth",
        "brand": "Carpano Antica",
        "abv": 0.165,
        "size_oz": 1.0
      }
    ]
  }
}
```

### Ingredient component fields

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | ✅ | Ingredient name |
| `abv` | float | ✅ | ABV as decimal |
| `size_oz` | float | ✅ | Volume in oz |
| `brand` | string | ❌ | Brand name if relevant |
| `published` | boolean | ❌ | false = proprietary, hide from display |
| `notes` | string | ❌ | Optional notes |
| `color` | object | ❌ | Color for cocktail builder mixing |

---

## Nutrition Object

```json
{
  "nutrition": {
    "calories": 195,
    "carbs_g": 17,
    "per_oz": false
  }
}
```

| Field | Type | Description |
|---|---|---|
| `calories` | integer | Calories per serving |
| `carbs_g` | float | Carbohydrates in grams per serving |
| `per_oz` | boolean | If true, values are per oz (multiply by serving size) |

---

## x-pound Vendor Extension

Pound-specific display and UX data. Ignored by other DrinkML clients.

```json
{
  "x-pound": {
    "icon_name": "rocks",
    "sound_name": "pour_whiskey",
    "category_display": "Cocktails",
    "default_pour_oz": 2.5,
    "search_tags": ["negroni", "campari", "gin", "classic"]
  }
}
```

| Field | Type | Description |
|---|---|---|
| `icon_name` | string | Asset catalog name for drink icon |
| `sound_name` | string | Sound file name for drink log sound |
| `category_display` | string | Display category in Pound's drink picker |
| `default_pour_oz` | float | Default serving size shown in picker |
| `search_tags` | array | Additional search terms |

---

## Complete Examples

### Beer
```json
{
  "drinkml": "1.0.0",
  "id": "org.drinkml:ipa@v1",
  "name": "IPA",
  "subtitle": "India Pale Ale",
  "category": "beer",
  "abv": 0.065,
  "serving": {
    "size_oz": 16,
    "glass": "shaker_pint"
  },
  "color": {
    "hex": "#C47A1E",
    "opacity": 0.88
  },
  "nutrition": {
    "calories": 195,
    "carbs_g": 17
  },
  "ingredients": {
    "disclosure": "none"
  },
  "tags": ["hoppy", "craft", "bitter"],
  "x-pound": {
    "icon_name": "pint",
    "sound_name": "pour_beer",
    "category_display": "Beer",
    "default_pour_oz": 16,
    "search_tags": ["ipa", "india pale ale", "hoppy"]
  }
}
```

### Wine
```json
{
  "drinkml": "1.0.0",
  "id": "org.drinkml:pinot-noir@v1",
  "name": "Pinot Noir",
  "subtitle": "Light Red Wine",
  "category": "wine",
  "abv": 0.130,
  "serving": {
    "size_oz": 5,
    "glass": "burgundy",
    "notes": "Standard 5oz pour. Adjust for your glass size."
  },
  "color": {
    "hex": "#8B3A3A",
    "opacity": 0.82
  },
  "nutrition": {
    "calories": 125,
    "carbs_g": 4
  },
  "ingredients": {
    "disclosure": "none"
  },
  "tags": ["red", "light", "elegant"],
  "x-pound": {
    "icon_name": "wine-glass-red",
    "sound_name": "pour_wine",
    "category_display": "Wine",
    "default_pour_oz": 5,
    "search_tags": ["pinot", "pinot noir", "light red"]
  }
}
```

### Cocktail (full ingredients)
```json
{
  "drinkml": "1.0.0",
  "id": "org.drinkml:negroni@v1",
  "name": "Negroni",
  "subtitle": "Equal parts classic",
  "category": "cocktail",
  "abv": 0.24,
  "serving": {
    "size_oz": 3.0,
    "glass": "rocks",
    "notes": "Served over a large ice cube with orange peel"
  },
  "color": {
    "hex": "#9B2335",
    "opacity": 0.92
  },
  "nutrition": {
    "calories": 180,
    "carbs_g": 4
  },
  "ingredients": {
    "disclosure": "full",
    "components": [
      {
        "name": "Gin",
        "brand": "Tanqueray",
        "abv": 0.473,
        "size_oz": 1.0,
        "color": {"hex": "#F5F5F0", "opacity": 0.05}
      },
      {
        "name": "Campari",
        "brand": "Campari",
        "abv": 0.250,
        "size_oz": 1.0,
        "color": {"hex": "#9B2335", "opacity": 0.95}
      },
      {
        "name": "Sweet Vermouth",
        "brand": "Carpano Antica",
        "abv": 0.165,
        "size_oz": 1.0,
        "color": {"hex": "#6B1818", "opacity": 0.85}
      }
    ]
  },
  "tags": ["classic", "bitter", "stirred", "iba"],
  "x-pound": {
    "icon_name": "rocks",
    "sound_name": "pour_cocktail",
    "category_display": "Cocktails",
    "default_pour_oz": 3.0,
    "search_tags": ["negroni", "campari", "gin", "bitter", "classic"]
  }
}
```

### Spirit
```json
{
  "drinkml": "1.0.0",
  "id": "org.drinkml:bourbon-rocks@v1",
  "name": "Bourbon",
  "subtitle": "On the rocks",
  "category": "spirit",
  "abv": 0.40,
  "serving": {
    "size_oz": 1.5,
    "glass": "rocks"
  },
  "color": {
    "hex": "#D4843A",
    "opacity": 0.85
  },
  "nutrition": {
    "calories": 105,
    "carbs_g": 0
  },
  "ingredients": {
    "disclosure": "none"
  },
  "tags": ["whiskey", "american", "aged"],
  "x-pound": {
    "icon_name": "rocks",
    "sound_name": "pour_whiskey",
    "category_display": "Spirits",
    "default_pour_oz": 1.5,
    "search_tags": ["bourbon", "whiskey", "rocks"]
  }
}
```

### Hard Seltzer
```json
{
  "drinkml": "1.0.0",
  "id": "org.drinkml:hard-seltzer@v1",
  "name": "Hard Seltzer",
  "subtitle": "White Claw, Truly, High Noon",
  "category": "beer",
  "abv": 0.050,
  "serving": {
    "size_oz": 12,
    "glass": "beer_can"
  },
  "color": {
    "hex": "#C8E8F5",
    "opacity": 0.15
  },
  "nutrition": {
    "calories": 100,
    "carbs_g": 2
  },
  "ingredients": {
    "disclosure": "none"
  },
  "tags": ["seltzer", "light", "sessionable", "gluten-free"],
  "x-pound": {
    "icon_name": "beer-can",
    "sound_name": "crack_can",
    "category_display": "Beer",
    "default_pour_oz": 12,
    "search_tags": ["seltzer", "hard seltzer", "white claw", "truly", "high noon", "mom water"]
  }
}
```

---

## Pace Calculation

Pound calculates alcohol grams from the DrinkML record:

```swift
// For simple drinks (disclosure: none)
let alcoholGrams = serving.size_oz * abv * 29.57 * 0.789

// For cocktails with full ingredients
let alcoholGrams = ingredients.components
    .filter { $0.published != false }
    .reduce(0) { total, component in
        total + (component.size_oz * component.abv * 29.57 * 0.789)
    }
```

One standard US drink = 14g pure alcohol = approximately 1.5oz at 40% ABV.

---

## Versioning

This is v1.0.0. Unknown fields are always ignored — forward compatibility by default. Breaking changes increment the major version. New optional fields increment the minor version.

---

## What's Deferred to v2+

- Tap system
- Color mixing algorithm
- Botanical profiles
- ABV history and market variants
- Bottled in Bond designation
- Cocktail `base` variation field
- Localization and translations
- Carbonation fields
- Brewing techniques enum
- Hopping schedule
- Vintage variation
- Generic trademark substitution model
- Registry and namespace verification infrastructure
- Personal ingredient sensitivity (Pound user profile feature — not spec)

---

*DrinkML is a Last Call Labs initiative. drinkml.org*
