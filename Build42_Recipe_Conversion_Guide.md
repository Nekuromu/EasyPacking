# Build 41 → Build 42 Recipe Conversion Guide for EasyPacking

## Executive Summary
**Major Change:** Build 42 completely rewrote the recipe system from `recipe` to `craftRecipe` with new structure, syntax, and requirements.

---

## 1. SYNTAX CONVERSION TABLE

| Build 41 | Build 42 |
|----------|----------|
| `recipe RuleName` | `craftRecipe RuleName` |
| `ItemName=Count,` | `item Count ItemName,` (in inputs/outputs blocks) |
| `Result:ItemName,` | Inside `outputs { item 1 ItemName, }` |
| `Time:60.0,` | `time = 60,` |
| `Category:Storage,` | `category = Storage,` |
| `OnTest:Function,` | `OnTest = Function,` |
| `OnCreate:Function,` | `OnCreate = Function,` |
| *(none)* | `tags = TagName;TagName,` **(REQUIRED - NEW)**|
| *(none)* | Must wrap inputs in `inputs { ... }` block |
| *(none)* | Must wrap outputs in `outputs { ... }` block |

---

## 2. DETAILED BEFORE/AFTER EXAMPLES

### Example 1: Simple Pack/Unpack (Bandages)

**Build 41:**
```
recipe Pack 9
{
    Bandage=9,
    
    Result:OS9pkBandages,
    OnTest:Recipe.OnTest.IsFavorite,
    Time:60.0,
    Category:Storage,
}

recipe Unpack 9
{
    OS9pkBandages,
    
    Result:Bandage=9,
    Time:60.0,
    Category:Storage,
}
```

**Build 42:**
```
craftRecipe PackBandages9
{
    tags = AnySurfaceCraft,
    time = 60,
    category = Storage,
    OnTest = Recipe.OnTest.IsFavorite,
    inputs
    {
        item 9 [Base.Bandage],
    }
    outputs
    {
        item 1 GidOrganized.OS9pkBandages,
    }
}

craftRecipe UnpackBandages9
{
    tags = AnySurfaceCraft,
    time = 60,
    category = Storage,
    inputs
    {
        item 1 [GidOrganized.OS9pkBandages],
    }
    outputs
    {
        item 9 Base.Bandage,
    }
}
```

---

### Example 2: Special Recipe with OnCreate (Skill Books)

**Build 41:**
```
recipe Unpack Carpentry Skill Books
{
    pkCarpentry,
    
    Result:BookCarpentry1,
    OnCreate:Recipe.OnCreate.UnpackCarpentrySkillBook,
    Time:100.0,
    Category:Storage,
}
```

**Build 42:**
```
craftRecipe UnpackCarpentrySkillBooks
{
    tags = AnySurfaceCraft,
    time = 100,
    category = Storage,
    OnCreate = Recipe.OnCreate.UnpackCarpentrySkillBook,
    inputs
    {
        item 1 [Packing.pkCarpentry],
    }
    outputs
    {
        item 1 Base.BookCarpentry1,
    }
}
```

---

### Example 3: With SaveUses Callback (Drainable Items)

**Build 41:**
```
recipe Pack 4
{
    ToiletPaper=4,
    
    Result:4pkToiletPaper,
    OnCreate:Recipe.OnCreate.SaveUses,
    OnTest:Recipe.OnTest.IsFavorite,
    Time:50.0,
    Category:Storage,
}
```

**Build 42:**
```
craftRecipe PackToiletPaper4
{
    tags = AnySurfaceCraft,
    time = 50,
    category = Storage,
    OnCreate = Recipe.OnCreate.SaveUses,
    OnTest = Recipe.OnTest.IsFavorite,
    inputs
    {
        item 4 [Base.ToiletPaper],
    }
    outputs
    {
        item 1 Packing.4pkToiletPaper,
    }
}
```

---

## 3. KEY STRUCTURAL CHANGES

### Old Format (Build 41)
```
recipe NAME
{
    INPUT=COUNT,
    INPUT2=COUNT2,
    
    Result:RESULT,
    Time:60.0,
    Category:Storage,
    OnCreate:Lua.Function,
}
```

### New Format (Build 42)
```
craftRecipe NAME
{
    tags = TagName,                    /* MANDATORY */
    time = 60,                         /* lowercase now */
    category = Storage,                /* lowercase now */
    OnCreate = Lua.Function,           /* = instead of : */
    
    inputs                             /* NEW: required block */
    {
        item COUNT [ItemID],
        item COUNT ItemID,
    }
    
    outputs                            /* NEW: required block */
    {
        item COUNT OutputID,
    }
}
```

---

## 4. REQUIRED TAGS (MANDATORY)

Every Build 42 craftRecipe MUST have at least ONE tag. Common tags for your mod:

| Tag | Use Case |
|-----|----------|
| `AnySurfaceCraft` | Can craft on any surface (ground, workbench, etc.) |
| `InHandCraft` | Can craft in-hand without a surface |
| `CanBeDoneFromFloor` | Can craft items from floor |
| `CanBeDoneInDark` | Can craft in darkness |

**For EasyPacking:** Use `AnySurfaceCraft` for pack/unpack (like storage) or `InHandCraft` if you prefer in-hand only.

Example:
```
tags = AnySurfaceCraft,       /* pack on any surface */
tags = InHandCraft,           /* unpack in hand */
tags = AnySurfaceCraft;InHandCraft,  /* both; use semicolon to separate */
```

---

## 5. LUA CALLBACK COMPATIBILITY ✅

**GOOD NEWS:** Your Lua callbacks ARE compatible with Build 42!

### Compatible Callbacks:
- ✅ `OnCreate` - Called when recipe completes
- ✅ `OnTest` - Validates if recipe can be performed
- ✅ Lua function signatures unchanged

### Your Lua Functions (Status):
- ✅ `Recipe.OnCreate.SaveUses` - **WORKS** (for drainables with durability)
- ✅ `Recipe.OnCreate.LoadUses` - **WORKS**
- ✅ `Recipe.OnCreate.UnpackCarpentrySkillBook` - **WORKS**
- ✅ `Recipe.OnTest.IsFavorite` - **WORKS**
- ✅ `Recipe.OnTest.WholeFood` - **WORKS**
- ✅ `Recipe.OnTest.WholeItem` - **WORKS**

### Potential Issues:
⚠️ The Lua function `saveItemAmounts()` in `extra_pack_functions.lua` uses `scriptManager:getAllRecipes()` which works with **Build 41 `Recipe` objects**. In Build 42, you'll need to use `scriptManager:getAllCraftRecipes()` instead.

**Change needed in Lua:**
```lua
-- OLD (Build 41)
local recipes = scriptManager:getAllRecipes()

-- NEW (Build 42)
local recipes = scriptManager:getAllCraftRecipes()
```

---

## 6. CONVERSION STEPS FOR YOUR MOD

1. **Backup** existing recipe files (already in common/ folder)

2. **Update all .txt files** in `Contents/mods/EasyPacking/media/scripts/PackingItems/`:
   - Change `recipe` → `craftRecipe`
   - Change `:` to `=` for all parameters
   - Wrap inputs with `inputs { item COUNT [ItemID], }`
   - Wrap outputs with `outputs { item COUNT ItemID, }`
   - Add `tags = AnySurfaceCraft,` (or appropriate tag)

3. **Update Lua file** (`extra_pack_functions.lua`):
   - Change `scriptManager:getAllRecipes()` → `scriptManager:getAllCraftRecipes()`
   - Update `recipe:getLuaCreate()` → appropriate Build 42 equivalent (if needed)
   - Update `recipe:getSource()` → appropriate Build 42 method (if needed)

4. **Test** with Project Zomboid Build 42

---

## 7. RECIPE ID NAMING

⚠️ **Important:** Recipe IDs cannot have spaces in Build 42!

| Build 41 | Build 42 |
|----------|----------|
| `recipe Pack 9` | `craftRecipe PackBandages9` |
| `recipe Unpack Carpentry Skill Books` | `craftRecipe UnpackCarpentrySkillBooks` |

Use camelCase or underscores, no spaces.

---

## 8. TRANSLATION FILES

Recipe names are now tied to translation files. For each recipe ID:

**File:** `media/lua/shared/Translate/EN/Recipes_EN.txt`

```lua
Recipes_EN = {
    Recipe_PackBandages9 = "Pack 9 Bandages",
    Recipe_UnpackBandages9 = "Unpack 9 Bandages",
    Recipe_UnpackCarpentrySkillBooks = "Unpack Carpentry Books",
}
```

The game will automatically look for `Recipe_{CraftRecipeID}` in translation files.

---

## Complete Working Example (Ready to Use)

```lua
module Packing
{
    imports { Base, }

    item 9pkBandages
    {
        Weight = 0.45,
        Type = Normal,
        DisplayName = 9 x Bandages,
        DisplayCategory = FirstAid,
        Icon = Bandages,
        WorldStaticModel = WIP,
    }

    craftRecipe PackBandages9
    {
        tags = AnySurfaceCraft,
        time = 60,
        category = Storage,
        OnTest = Recipe.OnTest.IsFavorite,
        inputs
        {
            item 9 [Base.Bandage],
        }
        outputs
        {
            item 1 Packing.9pkBandages,
        }
    }

    craftRecipe UnpackBandages9
    {
        tags = AnySurfaceCraft,
        time = 60,
        category = Storage,
        inputs
        {
            item 1 [Packing.9pkBandages],
        }
        outputs
        {
            item 9 Base.Bandage,
        }
    }
}
```

---

## Useful Resources
- **Wiki:** https://pzwiki.net/wiki/CraftRecipe
- **Scripts Syntax:** https://pzwiki.net/wiki/Scripts
- **Lua API for recipes:** https://pzwiki.net/wiki/Lua_(API)

