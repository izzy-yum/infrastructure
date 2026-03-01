# Complete Plate - Implementation Plan
**Based on Design Decisions - March 1, 2026**

---

## ✅ **Design Decisions Finalized**

### 1. **Side Selection: Optional (Suggested Workflow)**
- Users CAN skip grain/vegetable selection
- App gently suggests completing the plate
- No pressure, just helpful guidance
- Incomplete plate indicator shows what's missing

### 2. **Plate Indicator: Navbar Integration**
- Built into navigation bar
- Always visible at top of page
- Space-efficient design
- Works well on mobile

### 3. **Selection Flow: Hybrid (Recommendations + Browse)**
- Show 3-5 curated suggestions first
- "See all options" button for full grid
- Quick for users who trust recommendations
- Flexible for users who want to explore

### 4. **Vegetables: Multiple Allowed**
- Can add salad + roasted vegetables
- Better nutrition and variety
- More flexible meal planning
- Green section can show "2 veg" or expand to show both

---

## 🎯 Implementation Strategy

### **Phase 1: Foundation (Week 1)**
Build the basic infrastructure without disrupting existing app

**Tasks:**
1. Create database schema for side dishes
2. Add plate state management
3. Build PlateIndicator component (navbar)
4. Seed 5-10 grains
5. Seed 10-15 vegetables

**Goal:** Data and components ready, not yet integrated into flow

---

### **Phase 2: Grain Selection (Week 2)**
Add first side selection step

**Tasks:**
1. Build grain selection screen (hybrid model)
2. Update navigation after recipe selection
3. Add "Skip" option
4. Update plate indicator when grain selected
5. Test grain flow

**Goal:** Protein → Grain selection working

---

### **Phase 3: Vegetable Selection (Week 2-3)**
Add second side selection step

**Tasks:**
1. Build vegetable selection screen (hybrid model)
2. Support multiple vegetable selection
3. Update plate indicator for vegetables
4. Add "Skip" option
5. Test complete selection flow

**Goal:** Full selection flow working (Protein → Grain → Veg)

---

### **Phase 4: Complete Meal View (Week 3)**
Show the finished plate

**Tasks:**
1. Build CompleteMeal summary component
2. Display all 3+ components
3. Combined shopping list
4. Simple cooking timeline
5. Individual recipe access

**Goal:** Users see their complete balanced meal

---

### **Phase 5: Smart Features (Week 4)**
Add intelligence

**Tasks:**
1. Cuisine-based pairing recommendations
2. Shopping list consolidation (merge shared ingredients)
3. Cooking timeline optimization
4. Polish and animations

**Goal:** App intelligently guides users to good pairings

---

## 📐 Plate Indicator - Navbar Integration

### **Visual Design**

```
┌──────────────────────────────────────────────────────┐
│  [🍽️ Plate]  Izzy Yum       [Sign in] [Sign up]     │
└──────────────────────────────────────────────────────┘
    ↑
    Clickable plate icon shows:
    - Green wedge (vegetables)
    - Yellow wedge (grain)
    - Red wedge (protein)
    - Checkmarks on completed sections
```

**Sizes:**
- Desktop: 48x48px in navbar
- Mobile: 40x40px in navbar
- Click opens meal summary modal/dropdown

**States:**
- Empty: Gray outline, all sections empty
- 1 filled: Protein section colored with ✓
- 2 filled: Protein + Grain colored with ✓
- 3+ filled: All sections colored, subtle glow

### **Navbar Layout**

```tsx
<nav>
  <div className="flex items-center gap-4">
    <PlateIndicator
      protein={currentMeal.protein}
      grain={currentMeal.grain}
      vegetables={currentMeal.vegetables}
      onClick={openMealSummary}
    />
    <Link href="/">Izzy Yum</Link>
  </div>

  <div>
    <Link href="/login">Sign in</Link>
    <Link href="/register">Sign up</Link>
  </div>
</nav>
```

---

## 🗄️ Database Schema

### **New Tables**

```sql
-- Dish categories (Grain, Vegetable, Protein)
CREATE TABLE dish_categories (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  description TEXT,
  color TEXT, -- For UI: '#4ade80' for veg, '#fbbf24' for grain
  sort_order INTEGER
);

-- Side dishes (grains and vegetables)
CREATE TABLE side_dishes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  description TEXT,
  category_id UUID REFERENCES dish_categories(id),
  cuisine_id UUID REFERENCES cuisines(id), -- Can be NULL for universal sides
  image_url TEXT,
  prep_time_min INTEGER,
  cook_time_min INTEGER,
  total_time_min INTEGER,
  difficulty TEXT CHECK (difficulty IN ('Easy', 'Medium', 'Hard')),
  default_servings INTEGER DEFAULT 4,
  pairing_notes TEXT, -- "Pairs well with spicy dishes", etc.
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Side dish equipment
CREATE TABLE side_dish_equipment (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  side_dish_id UUID REFERENCES side_dishes(id) ON DELETE CASCADE,
  equipment TEXT NOT NULL,
  category TEXT,
  sort_order INTEGER
);

-- Side dish ingredients
CREATE TABLE side_dish_ingredients (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  side_dish_id UUID REFERENCES side_dishes(id) ON DELETE CASCADE,
  ingredient TEXT NOT NULL,
  amount DECIMAL,
  unit TEXT,
  category TEXT, -- Produce, Pantry, etc.
  notes TEXT,
  sort_order INTEGER
);

-- Side dish instructions
CREATE TABLE side_dish_instructions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  side_dish_id UUID REFERENCES side_dishes(id) ON DELETE CASCADE,
  phase TEXT CHECK (phase IN ('mise_en_place', 'cooking', 'plating')),
  step_number INTEGER,
  instruction TEXT NOT NULL,
  has_quantity BOOLEAN DEFAULT FALSE,
  clean_as_you_go TEXT
);

-- Optional: Meal combinations (for saving complete plates)
CREATE TABLE meals (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT, -- "My Thai Dinner", optional user-provided
  recipe_id UUID REFERENCES recipes(id),
  servings INTEGER,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Meal side dishes (many-to-many for multiple vegetables)
CREATE TABLE meal_sides (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  meal_id UUID REFERENCES meals(id) ON DELETE CASCADE,
  side_dish_id UUID REFERENCES side_dishes(id),
  quantity INTEGER DEFAULT 1, -- If user wants double portion of veg
  sort_order INTEGER
);
```

---

## 🌾 Initial Grain/Starch Data

### **Universal Grains (Work with Any Cuisine)**

1. **White Rice** - Easy, 20 min, pairs with everything
2. **Brown Rice** - Medium, 40 min, healthier option
3. **Quinoa** - Easy, 20 min, higher protein
4. **Roasted Potatoes** - Easy, 30 min, comfort food

### **Cuisine-Specific Grains**

**Thai/Vietnamese/Chinese:**
5. **Jasmine Rice** - Easy, 15 min
6. **Coconut Rice** - Easy, 20 min
7. **Rice Noodles** - Easy, 10 min

**Italian:**
8. **Polenta** - Easy, 25 min
9. **Gluten-Free Pasta** - Easy, 12 min

**Mexican:**
10. **Cilantro Lime Rice** - Easy, 20 min
11. **Corn Tortillas (warmed)** - Easy, 5 min

**Mediterranean:**
12. **Couscous** (GF) - Easy, 10 min

**American:**
13. **Sweet Potato** - Easy, 25 min
14. **Cornbread** - Medium, 25 min

---

## 🥦 Initial Vegetable Data

### **Universal Vegetables**

**Salads (Cold):**
1. **Mixed Green Salad** - Easy, 5 min
2. **Simple Cucumber Salad** - Easy, 10 min
3. **Tomato Salad** - Easy, 10 min

**Steamed:**
4. **Steamed Broccoli** - Easy, 8 min
5. **Steamed Green Beans** - Easy, 8 min
6. **Steamed Asparagus** - Easy, 6 min

**Roasted:**
7. **Roasted Brussels Sprouts** - Easy, 25 min
8. **Roasted Cauliflower** - Easy, 25 min
9. **Roasted Carrots** - Easy, 25 min

**Sautéed:**
10. **Sautéed Spinach** - Easy, 5 min
11. **Sautéed Zucchini** - Easy, 10 min

### **Cuisine-Specific Vegetables**

**Thai/Asian:**
12. **Thai Cucumber Salad** - Easy, 10 min
13. **Stir-Fried Bok Choy** - Easy, 8 min
14. **Thai Papaya Salad** - Medium, 15 min

**Mediterranean:**
15. **Greek Salad** - Easy, 10 min
16. **Roasted Mediterranean Vegetables** - Medium, 30 min

**Mexican:**
17. **Pico de Gallo** - Easy, 10 min
18. **Grilled Corn** - Easy, 15 min

---

## 🎨 PlateIndicator Component Spec

### **Component Props**

```tsx
interface PlateIndicatorProps {
  protein?: {
    name: string
    slug: string
  } | null
  grain?: {
    name: string
    slug: string
  } | null
  vegetables?: Array<{
    name: string
    slug: string
  }>
  size?: 'small' | 'medium' | 'large'
  onClick?: () => void
  showTooltip?: boolean
}
```

### **Visual States**

```tsx
// Empty state
<PlateIndicator />

// Protein only
<PlateIndicator protein={{ name: "Green Curry Shrimp", slug: "..." }} />

// Protein + Grain
<PlateIndicator
  protein={{ name: "Green Curry Shrimp", slug: "..." }}
  grain={{ name: "Jasmine Rice", slug: "..." }}
/>

// Complete plate
<PlateIndicator
  protein={{ name: "Green Curry Shrimp", slug: "..." }}
  grain={{ name: "Jasmine Rice", slug: "..." }}
  vegetables={[{ name: "Cucumber Salad", slug: "..." }]}
/>

// Complete with multiple vegetables
<PlateIndicator
  protein={{ name: "Green Curry Shrimp", slug: "..." }}
  grain={{ name: "Jasmine Rice", slug: "..." }}
  vegetables={[
    { name: "Cucumber Salad", slug: "..." },
    { name: "Bok Choy", slug: "..." }
  ]}
/>
```

---

## 🔄 Updated User Flow

### **Step-by-Step with Skip Options**

```
1. Home Page (Protein Grid)
   ↓
   User clicks: Beef
   Plate: [  ] [  ] [✓] (Protein selected)

2. Cuisine Selection
   ↓
   User clicks: Italian

3. Recipe Discovery
   ↓
   User clicks: Chicken Parmesan

4. Recipe Detail + Serving Selection
   ↓
   User selects: 4 servings

5. Transition Screen (NEW)
   ┌─────────────────────────────────────┐
   │ Complete Your Plate? (Optional)     │
   │                                     │
   │ Add a grain to balance your meal    │
   │                                     │
   │ [Continue to Add Grain]             │
   │ [Skip - Just View Recipe]           │
   └─────────────────────────────────────┘

   If skip → Go to Recipe Detail (existing)
   If continue → Go to Grain Selection

6. Grain Selection (NEW)
   Shows: 3-5 suggestions + "See all grains" button
   ↓
   User selects: Polenta
   Plate: [  ] [✓] [✓] (Grain + Protein)

7. Transition Screen (NEW)
   ┌─────────────────────────────────────┐
   │ Almost Done!                        │
   │                                     │
   │ Add vegetables to complete plate    │
   │                                     │
   │ [Continue to Add Vegetables]        │
   │ [Skip - View Meal Now]              │
   └─────────────────────────────────────┘

8. Vegetable Selection (NEW)
   Shows: 3-5 suggestions + "See all vegetables"
   Can select multiple (checkboxes)
   ↓
   User selects: Roasted Broccoli
   Plate: [✓] [✓] [✓] (Complete!)

9. Complete Meal View (NEW)
   Shows all components + cooking timeline
   ↓
   User clicks: "Send Shopping List to iPhone"
```

---

## 🛠️ Implementation Order (Recommended)

### **Sprint 1: Database & Foundation (2-3 days)**

**Tasks:**
1. ✅ Create database migration for new tables
2. ✅ Seed dish_categories (Grain, Vegetable)
3. ✅ Create 10 grain side dishes
4. ✅ Create 15 vegetable side dishes
5. ✅ Add to Supabase Cloud

**Deliverable:** Side dishes in database, ready to query

---

### **Sprint 2: Plate Indicator (1-2 days)**

**Tasks:**
1. ✅ Create PlateIndicator SVG component
2. ✅ Add to Navbar component
3. ✅ Create meal state management (Context or useState)
4. ✅ Test visual states (empty, partial, complete)

**Deliverable:** Plate indicator visible in navbar (shows protein only for now)

---

### **Sprint 3: Grain Selection (2-3 days)**

**Tasks:**
1. ✅ Create grain selection page (`/meal/grain`)
2. ✅ Build hybrid UI (suggestions + browse)
3. ✅ Add navigation after recipe selection
4. ✅ Add "Skip" option
5. ✅ Update plate indicator when grain selected
6. ✅ Store grain selection in state

**Deliverable:** Grain selection working, plate updates

---

### **Sprint 4: Vegetable Selection (2-3 days)**

**Tasks:**
1. ✅ Create vegetable selection page (`/meal/vegetables`)
2. ✅ Build hybrid UI with multi-select
3. ✅ Add navigation after grain selection (or after recipe if skipped)
4. ✅ Update plate indicator for vegetables
5. ✅ Store vegetable selection(s) in state

**Deliverable:** Full selection flow working

---

### **Sprint 5: Complete Meal View (3-4 days)**

**Tasks:**
1. ✅ Create CompleteMeal page/component
2. ✅ Display all selected components
3. ✅ Build consolidated shopping list
4. ✅ Build simple cooking timeline
5. ✅ Add edit options (change grain/vegetables)
6. ✅ Integrate with existing shopping list panel

**Deliverable:** Users see complete balanced meal with timeline

---

### **Sprint 6: Polish & Smart Features (3-4 days)**

**Tasks:**
1. ✅ Cuisine-based recommendations algorithm
2. ✅ Shopping list smart consolidation
3. ✅ Timeline optimization (parallel cooking)
4. ✅ Animations and transitions
5. ✅ Mobile responsive design
6. ✅ Testing and bug fixes

**Deliverable:** Polished, intelligent meal planning system

---

## 🎨 Component Architecture

### **New Components**

```
src/components/
├── PlateIndicator.tsx          (SVG plate with 3 sections)
├── SideSelection.tsx           (Reusable for grain/veg)
├── SideCard.tsx                (Like RecipeCard but for sides)
├── CompleteMeal.tsx            (Shows full meal)
├── CookingTimeline.tsx         (Sequences dishes)
├── MealSummaryModal.tsx        (Click plate to view)
└── ConsolidatedShoppingList.tsx (Merged ingredients)

src/app/
├── meal/
│   ├── grain/page.tsx          (Grain selection)
│   ├── vegetables/page.tsx     (Vegetable selection)
│   └── summary/page.tsx        (Complete meal view)

src/lib/
├── mealState.ts                (State management)
├── pairingLogic.ts             (Recommendations)
├── shoppingConsolidation.ts    (Merge ingredients)
└── cookingTimeline.ts          (Sequence optimizer)
```

---

## 🔑 Meal State Management

### **Option A: React Context (Recommended)**

```tsx
// src/contexts/MealContext.tsx
interface MealContextType {
  protein: Recipe | null
  grain: SideDish | null
  vegetables: SideDish[]
  servings: number

  setProtein: (recipe: Recipe, servings: number) => void
  setGrain: (grain: SideDish) => void
  addVegetable: (veg: SideDish) => void
  removeVegetable: (vegId: string) => void
  clearMeal: () => void
  isComplete: boolean
}

// Usage
const { protein, grain, vegetables, setGrain } = useMeal()
```

### **Option B: URL State (Alternative)**

```
/meal/grain?protein=green-curry-shrimp&servings=4
/meal/vegetables?protein=...&servings=4&grain=jasmine-rice
/meal/summary?protein=...&grain=...&veg=cucumber-salad,bok-choy
```

**Pros:** Shareable URLs, no context needed
**Cons:** Long URLs, more complex

**Recommendation:** Use Context for better UX

---

## 🎯 MVP Features (Simplified for First Release)

### **What to Include:**

✅ **Must Have:**
- Plate indicator in navbar
- Optional grain selection (skip allowed)
- Optional vegetable selection (skip allowed)
- Complete meal view showing all components
- Combined shopping list

✅ **Should Have:**
- 10 grain options
- 15 vegetable options
- Cuisine-based suggestions (3-5 per type)
- Multiple vegetable selection
- Basic cooking timeline

❌ **Can Wait:**
- Smart ingredient consolidation (just list separately for v1)
- Advanced timeline optimization
- Saved favorite meals
- Pairing quality scores
- Nutritional breakdown

---

## 📊 Example: First Complete Meal

**Protein:** Green Curry Shrimp (Thai, 4 servings, 30 min)
**Grain:** Jasmine Rice (Universal, 4 servings, 15 min)
**Vegetable:** Cucumber Salad (Thai, 4 servings, 10 min)

**Shopping List (Simple v1 - no consolidation):**

```
From Green Curry Shrimp:
  Proteins:
  - 2 lbs shrimp
  Produce:
  - 2 red bell peppers
  - Fresh Thai basil
  Pantry:
  - 2 cans coconut milk
  - Green curry paste

From Jasmine Rice:
  Pantry:
  - 2 cups jasmine rice

From Cucumber Salad:
  Produce:
  - 2 cucumbers
  - 1 red onion
  - Fresh cilantro
  Pantry:
  - Rice vinegar
  - Sesame oil
```

**Cooking Timeline:**

```
6:20 PM - Start Jasmine Rice (15 min) → Ready 6:35 PM
6:30 PM - Make Cucumber Salad (10 min) → Ready 6:40 PM, refrigerate
6:35 PM - Start Green Curry (30 min) → Ready 7:05 PM
7:05 PM - Everything ready to serve!
```

---

## 🚀 Quick Start: What to Build First?

### **Option A: Database First (Recommended)**
**Why:** Can't build UI without data
**Time:** 1-2 days
**Tasks:**
1. Create migrations
2. Seed 10 grains + 15 vegetables
3. Deploy to Supabase Cloud
4. Verify data loads

### **Option B: Plate Indicator First**
**Why:** Visual progress, user can see it immediately
**Time:** 1 day
**Tasks:**
1. Build SVG component
2. Add to navbar
3. Connect to existing recipe selection
4. Test visual states

### **Option C: Both in Parallel**
**Why:** Fastest overall
**Tasks:**
- You work on side dish data/recipes
- I build PlateIndicator component
- Merge when both ready

---

## 💡 My Recommendation

**Start with:** Database + PlateIndicator in parallel

**Today:**
1. I'll create the database migrations
2. I'll build the PlateIndicator component
3. I'll seed 5 example grains and 5 example vegetables
4. Deploy to production

**Result:**
- Plate indicator visible in navbar
- Shows protein selection status
- Example side dishes in database
- Foundation ready for grain/veg selection screens

**Time:** 3-4 hours

**Then:** You can review the plate indicator and example sides, give feedback, and we continue building the selection screens.

---

**Sound good? Want me to start building the foundation (database + plate indicator)?**
