# Complete Plate Concept
## Balanced Gluten-Free Meal Planning

**Status:** Design Phase
**Created:** March 1, 2026
**Related:** Enhancement to core recipe planning system

---

## 🍽️ The Vision

Move from **single dish planning** (just protein) to **complete meal planning** (protein + grain + vegetable) with nutritionally balanced proportions.

### **Current State:**
User selects → Protein → Cuisine → Recipe
Result: One dish (e.g., Green Curry Shrimp)

### **Future State:**
User selects → Protein → Cuisine → Recipe → Grain/Starch → Vegetable
Result: Complete balanced meal (e.g., Green Curry Shrimp + Jasmine Rice + Cucumber Salad)

---

## 🥗 The Balanced Plate Concept

### **Nutritional Proportions (by volume)**

```
┌─────────────────────────────────────┐
│          COMPLETE PLATE             │
├─────────────────────────────────────┤
│                                     │
│   🥦 Vegetables:  50% (1/2 plate)   │
│   🌾 Grain/Starch: 25% (1/4 plate)  │
│   🍖 Protein:     25% (1/4 plate)   │
│                                     │
└─────────────────────────────────────┘
```

**Based on:**
- USDA MyPlate guidelines
- Celiac disease nutrition recommendations
- Balanced macronutrient distribution
- Easy-to-visualize portions

---

## 🎨 Visual Design: The Plate Indicator

### **Plate Icon (Upper Left Corner)**

```
  ┌─────────────────┐
  │     ┌─────┐     │
  │    ╱   🥦  ╲    │  ← Vegetables (Green) 50%
  │   │  ┌───┐  │   │
  │   │ 🌾│🍖 │  │   │  ← Grain (Yellow) 25% + Protein (Red) 25%
  │   │  └───┘  │   │
  │    ╲       ╱    │
  │     └─────┘     │
  └─────────────────┘

Visual representation:
- Circular plate divided into 3 sections
- Green wedge (top half): Vegetables
- Yellow wedge (bottom left quarter): Grain/Starch
- Red wedge (bottom right quarter): Protein
```

### **State Indicators**

**Empty Plate (Start):**
```
All sections: Light gray outline
No checkmarks
```

**Protein Selected:**
```
Red section: ✓ Checkmark appears
Yellow section: Still empty (pulsing to draw attention?)
Green section: Still empty
```

**Protein + Grain Selected:**
```
Red section: ✓ Green checkmark
Yellow section: ✓ Green checkmark
Green section: Still empty (pulsing)
```

**Complete Plate:**
```
Red section: ✓ Green checkmark
Yellow section: ✓ Green checkmark
Green section: ✓ Green checkmark
Entire plate: Subtle glow or success state
```

---

## 📖 Isabelle's Extended User Story

### Scene 1: Morning Planning (Original + Extended)

**Time:** 7:15 AM
**Location:** Isabelle's apartment, kitchen table

Isabelle opens her laptop to plan dinner. She notices the **plate indicator** in the upper left corner - three empty sections waiting to be filled.

**The plate shows:**
- 🥦 Vegetables (empty)
- 🌾 Grain (empty)
- 🍖 Protein (empty)

She's been craving seafood. She clicks **Shrimp** → **Thai** → **Green Curry Shrimp** (4 servings).

**The plate updates:**
- 🥦 Vegetables (empty)
- 🌾 Grain (empty)
- 🍖 Protein **✓** (Green Curry Shrimp selected)

The app transitions smoothly. Instead of going straight to the recipe detail, a new screen appears...

---

### Scene 2: Completing the Plate - Grain Selection

**Time:** 7:18 AM
**Location:** Still at her kitchen table

The screen shows a beautiful header:

**"Complete Your Plate"**
**"Add a grain or starch to balance your meal"**

Below, she sees a grid of grain/starch options that pair well with Thai cuisine:

- **Jasmine Rice** - "15 min • Easy • Traditional pairing"
- **Coconut Rice** - "20 min • Easy • Rich and aromatic"
- **Rice Noodles** - "10 min • Easy • Light and fresh"
- **Quinoa** - "20 min • Easy • Higher protein option"

Each card shows a beautiful photo, prep time, and a brief description.

She clicks **Jasmine Rice** (she loves the classic pairing).

**The plate updates:**
- 🥦 Vegetables (empty)
- 🌾 Grain **✓** (Jasmine Rice selected)
- 🍖 Protein **✓** (Green Curry Shrimp selected)

---

### Scene 3: Completing the Plate - Vegetable Selection

**Time:** 7:19 AM
**Location:** Still planning, excited about her balanced meal

The screen transitions again:

**"Almost Done!"**
**"Add vegetables to complete your plate"**

A grid of vegetable side dishes appears, curated for Thai cuisine:

- **Cucumber Salad** - "10 min • Easy • Cool and refreshing"
- **Stir-Fried Bok Choy** - "8 min • Easy • Light and crisp"
- **Thai Papaya Salad** - "15 min • Medium • Tangy and spicy"
- **Steamed Broccoli with Garlic** - "10 min • Easy • Simple and healthy"

She chooses **Cucumber Salad** - perfect for balancing the rich curry.

**The plate completes:**
- 🥦 Vegetables **✓** (Cucumber Salad)
- 🌾 Grain **✓** (Jasmine Rice)
- 🍖 Protein **✓** (Green Curry Shrimp)

The entire plate indicator glows with a subtle success animation.

---

### Scene 4: Complete Meal Recipe View

**Time:** 7:20 AM
**Location:** Reviewing her complete meal plan

The screen now shows her **Complete Meal** with all three components:

**Header:**
"Your Balanced Thai Meal" (4 servings)

**Meal Components:**
1. **Green Curry Shrimp** (Protein) - 30 min
2. **Jasmine Rice** (Grain) - 15 min
3. **Cucumber Salad** (Vegetable) - 10 min

**Total prep time:** 55 minutes (can be done in parallel!)

**Cooking Strategy (Auto-generated):**
```
Timeline (everything ready at 6:50 PM):

6:05 PM - Start Jasmine Rice (15 min)
6:20 PM - Start Cucumber Salad prep (10 min, can chill)
6:20 PM - Start Green Curry Shrimp (30 min)
6:50 PM - Everything ready to plate!

Most dishes can be prepped in parallel!
```

She sees three tabs or sections:
- **Shopping List** - Combined ingredients from all three dishes
- **Equipment** - All equipment needed
- **Cooking Instructions** - Organized by dish with timing

She clicks **"Send Shopping List to iPhone"**

---

### Scene 5: Shopping with a Complete Meal

**Time:** 5:45 PM
**Location:** Whole Foods

Her shopping list is organized by category, but now includes ingredients from **all three dishes**:

**Proteins:**
- ☐ 2 lbs large shrimp

**Produce:**
- ☐ 2 red bell peppers (curry)
- ☐ Fresh Thai basil (curry)
- ☐ 2 cucumbers (salad)
- ☐ 1 red onion (salad)
- ☐ Fresh cilantro (salad)
- ☐ 2 limes (curry + salad)

**Pantry:**
- ☐ 2 cans coconut milk
- ☐ Jasmine rice
- ☐ Green curry paste
- [etc...]

She loves that:
- Shared ingredients are consolidated (one "2 limes" instead of listing twice)
- Everything is organized by store section
- The list represents a **complete, balanced meal**

---

### Scene 6: Cooking a Complete Meal

**Time:** 6:05 PM
**Location:** Her kitchen

Isabelle opens her laptop to the meal view. She can see all three recipes with a **smart cooking timeline**:

**6:05 PM:**
- Start: Jasmine Rice (will be done at 6:20 PM)

**6:10 PM:**
- Prep: Cucumber Salad (make ahead, let it chill)

**6:20 PM:**
- Start: Green Curry Shrimp (main focus, 30 min)

**6:50 PM:**
- Plate: Everything ready at the same time!

The app helped her sequence the cooking so nothing sits around getting cold.

---

### Scene 7: A Balanced, Beautiful Meal

**Time:** 6:55 PM
**Location:** Her dining table

Isabelle looks at her plate:

- **Half the plate:** Crisp, refreshing cucumber salad
- **Quarter of the plate:** Fluffy, fragrant jasmine rice
- **Quarter of the plate:** Rich, aromatic green curry shrimp

It's not just delicious - it's **nutritionally balanced**, **visually beautiful**, and **gluten-free safe**.

She reflects on the experience:

**Old way (just protein):**
- Made Green Curry Shrimp
- Ate it with random sides from her fridge
- Imbalanced, not well-planned

**New way (complete plate):**
- Planned a full, balanced meal
- Everything coordinated and timed
- Nutritionally complete
- Shopping list covered everything
- Felt like a restaurant meal

This is what meal planning should be: **complete, balanced, and joyful**.

---

## 🎯 Key Design Principles

### 1. **Guided Progression**

**Current:** Protein → Cuisine → Recipe (3 steps)
**New:** Protein → Cuisine → Recipe → Grain → Vegetable (5 steps)

**Why it works:**
- Logical flow (main dish first, then sides)
- Builds excitement (completing the plate)
- Educational (teaches balanced eating)

### 2. **Visual Feedback**

The plate indicator provides:
- ✅ Progress tracking (how complete is my meal?)
- ✅ Visual representation of balance
- ✅ Gamification (satisfying to fill the plate)
- ✅ Educational tool (shows proper proportions)

### 3. **Smart Consolidation**

**Shopping List:**
- Shared ingredients merged (1 entry for "2 limes" instead of 2 separate)
- Still organized by category
- Still shows which recipe each ingredient is for (optional detail view)

**Equipment:**
- Combined list from all dishes
- No duplicates (one "cutting board" even if used in 3 recipes)

**Cooking Timeline:**
- Automatically sequences dishes
- Optimizes for everything ready at same time
- Shows parallel prep opportunities

### 4. **Flexibility**

Users can:
- Skip grain or vegetable if they want (plate shows as incomplete)
- Replace selections (click a different grain)
- View just the protein recipe (old behavior still available)
- Add multiple vegetables or grains (advanced)

---

## 🗂️ Database Schema Changes

### New Tables

**`dish_categories`**
```sql
id          UUID PRIMARY KEY
name        TEXT  -- "Grain/Starch", "Vegetable", "Protein"
slug        TEXT  -- "grain", "vegetable", "protein"
description TEXT
sort_order  INTEGER
```

**`side_dishes`**
```sql
id               UUID PRIMARY KEY
name             TEXT NOT NULL
slug             TEXT UNIQUE
description      TEXT
category_id      UUID REFERENCES dish_categories(id)  -- grain or vegetable
image_url        TEXT
prep_time_min    INTEGER
difficulty       TEXT
default_servings INTEGER DEFAULT 4
created_at       TIMESTAMP
```

**`side_dish_ingredients`** (similar to recipe_ingredients)
**`side_dish_instructions`** (similar to recipe_instructions)
**`side_dish_equipment`** (similar to recipe_equipment)

**`meal_pairings`** (Optional - for curated recommendations)
```sql
id          UUID PRIMARY KEY
recipe_id   UUID REFERENCES recipes(id)
grain_id    UUID REFERENCES side_dishes(id)
veg_id      UUID REFERENCES side_dishes(id)
pairing_quality TEXT  -- "excellent", "good", "acceptable"
notes       TEXT  -- Why this pairing works
```

---

## 🎨 UI Components

### 1. **Plate Indicator Component**

**Location:** Fixed in upper left corner (or top center)

**Specifications:**
- Size: 80x80px (desktop), 60x60px (mobile)
- Always visible while browsing
- Clickable to view meal summary
- Animates when sections complete

**States:**
```
Empty:     All sections gray outline
1 filled:  1 section colored with checkmark
2 filled:  2 sections colored with checkmarks
Complete:  All 3 sections colored, subtle glow animation
```

**Colors:**
- Green section (#4ade80): Vegetables
- Yellow section (#fbbf24): Grains/Starches
- Red section (#ef4444): Protein

### 2. **Side Dish Selection Screens**

**Similar to current recipe discovery but for sides:**
- Grid of side dish cards
- Photos, prep time, difficulty
- Filter by cuisine compatibility
- "Skip this" option

### 3. **Complete Meal View**

**New summary screen showing:**
- All 3 components with images
- Combined shopping list
- Cooking timeline
- Equipment needed
- Can edit any component

### 4. **Meal Shopping List**

**Enhanced shopping list with:**
- Ingredients from all dishes combined
- Smart consolidation (shared ingredients)
- Organized by category
- Optional: color-code by dish

---

## 🔄 Updated User Flow

### Flow A: Complete Plate Journey (New Default)

```
1. Home (Protein Grid)
   ↓ Select Protein

2. Cuisine Selection
   ↓ Select Cuisine

3. Recipe Discovery (Protein)
   ↓ Select Recipe

4. Serving Size Selection
   ↓ Select Servings

5. Grain Selection (NEW)
   ↓ Select Grain/Starch

6. Vegetable Selection (NEW)
   ↓ Select Vegetable

7. Complete Meal View (NEW)
   ├─ View combined recipe
   ├─ See cooking timeline
   ├─ Send shopping list
   └─ View individual dishes

Plate indicator updates at each step!
```

### Flow B: Quick Protein-Only (Existing Behavior)

```
User can:
- Click "Skip to Recipe" at any step
- View just protein recipe
- Add sides later (from meal view)
- Plate indicator shows incomplete
```

---

## 🧩 Side Dish Categories

### **Grains & Starches**

**Types:**
- **Rice varieties:** Jasmine, Basmati, Brown, Wild, Coconut Rice
- **Alternative grains:** Quinoa, Millet, Rice Pasta
- **Starches:** Roasted Potatoes, Sweet Potatoes, Polenta
- **Gluten-free breads:** Cornbread, GF Naan, GF Tortillas

**Considerations:**
- Must be gluten-free
- Prep time: 10-30 minutes
- Typically serves 4-6
- Some can be made ahead

### **Vegetables**

**Types:**
- **Salads:** Cucumber Salad, Mixed Greens, Tomato Salad
- **Roasted:** Roasted Broccoli, Brussels Sprouts, Cauliflower
- **Steamed:** Green Beans, Asparagus, Bok Choy
- **Sautéed:** Spinach, Zucchini, Mixed Vegetables
- **Raw:** Veggie Sticks with Dip, Coleslaw

**Considerations:**
- Must be gluten-free
- Prep time: 5-20 minutes
- Should complement main dish flavor profile
- Can be simple or elaborate

---

## 🤖 Smart Pairing Algorithm

### **Cuisine-Based Recommendations**

When user selects a Thai protein recipe:

**Recommended Grains:**
- ⭐ Jasmine Rice (traditional)
- ⭐ Coconut Rice (flavor enhancement)
- Rice Noodles (alternative)
- Quinoa (health-conscious)

**Recommended Vegetables:**
- ⭐ Cucumber Salad (cooling balance)
- ⭐ Stir-Fried Bok Choy (traditional)
- Thai Papaya Salad (authentic)
- Steamed Broccoli (simple)

**Stars (⭐) indicate "Excellent pairing"**

### **Flavor Profile Matching**

**If protein is:**
- **Rich/creamy** → Light, fresh vegetables
- **Spicy** → Cooling vegetables (cucumber, etc.)
- **Mild** → Bold, flavorful vegetables
- **Heavy** → Simple grains

**If grain is:**
- **Plain rice** → Can handle bold vegetables
- **Flavored rice** → Simpler vegetables
- **Heavy (polenta)** → Light vegetables

---

## 📊 Shopping List Consolidation

### **Smart Ingredient Merging**

**Scenario:** Green Curry Shrimp + Jasmine Rice + Cucumber Salad

**Before consolidation:**
```
Green Curry:
- 2 limes
- Fresh cilantro

Cucumber Salad:
- 1 lime
- Fresh cilantro
- 2 cucumbers
```

**After consolidation:**
```
Produce:
- 🥦 2 cucumbers (Cucumber Salad)
- 🍋 3 limes (2 for curry, 1 for salad)
- 🌿 Fresh cilantro (for curry and salad)
```

**Smart rules:**
- Same ingredient + same unit → combine amounts
- Show which dishes use shared ingredients (expandable)
- If amounts differ (bunch vs. tablespoon) → keep separate with dish names

---

## ⏰ Cooking Timeline Generation

### **Algorithm:**

**Input:**
- Protein: 30 min (Green Curry)
- Grain: 15 min (Jasmine Rice)
- Vegetable: 10 min (Cucumber Salad)

**Target:** All dishes ready at 6:50 PM

**Output:**
```
🕐 6:05 PM - Start Cucumber Salad (10 min)
   ├─ Can refrigerate until serving
   └─ Prep first since it can sit

🕐 6:20 PM - Start Jasmine Rice (15 min)
   ├─ Rice cooker can keep warm
   └─ Start 30 min before serving

🕐 6:35 PM - Start Green Curry Shrimp (30 min but flexible)
   ├─ Main dish, cook last for freshness
   └─ Can adjust timing by a few minutes

🕐 6:50 PM - Everything ready to plate!
```

**Factors considered:**
- Which dishes can sit (salads, rice)
- Which must be served immediately (proteins)
- Which can be prepped ahead (cold dishes)
- Equipment conflicts (if sharing stove/oven)

---

## 🎯 Implementation Phases

### **Phase 1: Design & Data (This Document)**
- ✅ Define the concept
- ✅ Create user story
- ✅ Design plate indicator
- Design database schema
- Plan UI components
- **Deliverable:** Design documentation (this file)

### **Phase 2: Database & Side Dishes**
- Create new tables (dish_categories, side_dishes, etc.)
- Seed initial grain options (10-15)
- Seed initial vegetable options (15-20)
- Create side dish data structure
- **Deliverable:** 25-35 side dishes in database

### **Phase 3: UI Components**
- Build PlateIndicator component
- Build SideSelection screen (reuse RecipeDiscovery pattern)
- Build CompleteMeal view
- Update navigation flow
- **Deliverable:** Working UI components

### **Phase 4: Smart Features**
- Shopping list consolidation logic
- Cooking timeline generator
- Cuisine-based pairing recommendations
- **Deliverable:** Intelligent meal planning

### **Phase 5: Polish**
- Animations for plate filling
- Improved cooking timeline UI
- Better pairing suggestions
- Optional: Save complete meals as favorites
- **Deliverable:** Polished experience

---

## 🎨 Plate Indicator - Technical Specs

### **SVG Component Structure**

```tsx
<PlateIndicator
  protein={selectedProtein}   // Recipe object or null
  grain={selectedGrain}       // Side dish object or null
  vegetable={selectedVeg}     // Side dish object or null
  size="large"                // "small" | "medium" | "large"
  interactive={true}          // Clickable to open meal summary
/>
```

### **Visual Design (SVG)**

```svg
<svg viewBox="0 0 100 100" className="plate-indicator">
  <!-- Plate background -->
  <circle cx="50" cy="50" r="45" fill="#f5f5f5" stroke="#e5e5e5" />

  <!-- Vegetable section (top half, 180°) -->
  <path
    d="M 50 50 L 50 5 A 45 45 0 0 1 95 50 Z"
    fill={vegetable ? "#4ade80" : "#d1d5db"}
    opacity={vegetable ? 1 : 0.3}
  />
  {vegetable && <text x="50" y="25" fontSize="20">✓</text>}

  <!-- Grain section (bottom left, 90°) -->
  <path
    d="M 50 50 L 5 50 A 45 45 0 0 0 50 95 Z"
    fill={grain ? "#fbbf24" : "#d1d5db"}
    opacity={grain ? 1 : 0.3}
  />
  {grain && <text x="30" y="75" fontSize="20">✓</text>}

  <!-- Protein section (bottom right, 90°) -->
  <path
    d="M 50 50 L 50 95 A 45 45 0 0 0 95 50 Z"
    fill={protein ? "#ef4444" : "#d1d5db"}
    opacity={protein ? 1 : 0.3}
  />
  {protein && <text x="70" y="75" fontSize="20">✓</text>}
</svg>
```

---

## 📐 Proportions & Serving Sizes

### **Volume-Based Portions**

For 1 person:
- Vegetables: ~2 cups (50%)
- Grain: ~1 cup (25%)
- Protein: ~4-6 oz or ~1 cup (25%)

For 4 servings:
- Vegetables: ~8 cups total
- Grain: ~4 cups total
- Protein: ~1-1.5 lbs or ~4 cups

**Important:** Proportions by **volume**, not weight (visual plating guide)

---

## 🔄 Side Dish Data Model

### **Simplified Side Dish**

Side dishes are simpler than main recipes:
- Usually single-component (just vegetables or just grain)
- Fewer ingredients (3-8 typically)
- Shorter prep time (5-20 min)
- No protein/cuisine classification needed
- Can be paired with multiple main dishes

**Example: Cucumber Salad**
```
Category: Vegetable
Cuisine affinity: Thai, Vietnamese, Japanese, Mediterranean
Ingredients: Cucumber, rice vinegar, sesame oil, salt
Time: 10 min
Serves: 4
```

---

## 🎯 Success Metrics

### **User Engagement**
- % of users who complete the full plate (vs. protein only)
- Time spent on side selection
- Most popular grain/vegetable combinations

### **Nutritional Impact**
- Average vegetable intake per meal planned
- Balance score (how close to 50/25/25)
- Variety of vegetables selected

### **Meal Planning Quality**
- Complete meals planned per week
- Shopping list accuracy
- User satisfaction with pairings

---

## 🚀 Future Enhancements

### **Phase 2+**
- **Multiple vegetables:** Add 2-3 vegetable sides
- **Soup/salad/bread:** Additional categories
- **Dessert:** Optional sweet finish
- **Beverage pairing:** Wine, tea, etc. suggestions
- **Leftover planning:** Scale individual components differently
- **Meal templates:** Pre-built balanced meals
- **Seasonal recommendations:** Based on time of year
- **Dietary customization:** Low-carb, high-protein, etc.

---

## 📊 Comparison: Current vs. Complete Plate

| Feature | Current (Protein Only) | Complete Plate |
|---------|----------------------|----------------|
| Selection steps | 3 (Protein → Cuisine → Recipe) | 5 (+ Grain + Vegetable) |
| Plate indicator | None | Visual progress tracker |
| Shopping list | One dish | Consolidated from 3 dishes |
| Cooking guidance | One recipe | Timeline for all dishes |
| Nutritional balance | User's responsibility | Guided to 50/25/25 |
| Time to plan | ~2 min | ~4-5 min |
| Meal completeness | Partial | Complete |
| Educational value | Low | High (teaches balance) |

---

## 💡 Design Decisions

### **Why Guide to Complete Plates?**

1. **Health:** Celiac patients need balanced nutrition
2. **Education:** Teaches proper meal composition
3. **Convenience:** Everything planned together
4. **Shopping:** One consolidated list
5. **Timing:** Coordinated cooking schedule

### **Why Make It Optional?**

1. **Flexibility:** Users can skip if desired
2. **Time:** Not everyone wants full meals
3. **Complexity:** Some want simple single-dish dinners
4. **Testing:** Can measure adoption rate

### **Why Start Simple?**

1. **MVP:** Get basic functionality working first
2. **Iteration:** Learn from user behavior
3. **Database:** Easier to expand than simplify
4. **Development:** Phase implementation reduces risk

---

## 🎯 Next Steps for Implementation

1. **[ ] Review this design document** - Get feedback
2. **[ ] Create detailed mockups** - Visual design of each screen
3. **[ ] Design database migrations** - Schema for side dishes
4. **[ ] Seed initial side dishes** - 10 grains, 15 vegetables
5. **[ ] Build PlateIndicator component** - Visual progress tracker
6. **[ ] Build grain selection screen** - First side selection
7. **[ ] Build vegetable selection screen** - Second side selection
8. **[ ] Build CompleteMeal view** - Final summary
9. **[ ] Implement shopping list consolidation** - Smart merging
10. **[ ] Build cooking timeline** - Sequence optimizer
11. **[ ] Test complete flow** - End-to-end
12. **[ ] Deploy and iterate** - Get user feedback

**Estimated effort:** 3-4 weeks for complete implementation

---

## ❓ Open Questions

1. **Should grain/vegetable be required?** Or optional?
2. **Can users add multiple vegetables?** (E.g., salad + roasted veg)
3. **Show plate indicator on every page?** Or only during selection?
4. **Allow swapping after completion?** (Replace grain without re-selecting protein)
5. **Pre-built meal combos?** Offer curated complete meals
6. **Cuisine filtering for sides?** Only show Thai vegetables with Thai protein?

---

*This concept transforms Izzy Yum from a recipe app to a complete meal planning system for balanced, gluten-free eating.*
