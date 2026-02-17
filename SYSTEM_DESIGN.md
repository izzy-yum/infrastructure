# Izzy Yum System Design
## Technical Architecture & Component Design

---

## Overview

This document describes the technical implementation of the Izzy Yum recipe system, mapping each element of Isabelle's journey to specific application components, data models, and interactions.

---

## Application Components

### Web Application (Next.js)

The web app is Isabelle's planning and cooking companion, used primarily on her laptop.

#### Core Screens

**1. Protein Selection Screen** (`/`)
- **Component**: `ProteinGrid`
- **Data**: Static array of protein objects with images and metadata
- **Interaction**: Click protein → navigate to `/cuisine?protein=shrimp`
- **Animation**: Framer Motion fade-out, slide-in transition

**2. Cuisine Selection Screen** (`/cuisine?protein={protein}`)
- **Component**: `CuisineGrid`
- **Data**: Static array of cuisine objects filtered by protein compatibility
- **Interaction**: Click cuisine → navigate to `/recipes?protein={protein}&cuisine=thai`
- **Animation**: Grid fade-out, recipe cards fade-in

**3. Recipe Discovery Screen** (`/recipes?protein={protein}&cuisine={cuisine}`)
- **Components**:
  - `RecipeCardGrid` - Main grid of recipe cards
  - `IngredientWheel` - Horizontal scrollable filter
- **Data**:
  - Recipes fetched from Supabase filtered by protein + cuisine
  - Secondary ingredients dynamically generated from matching recipes
- **Interaction**:
  - Click ingredient → filter recipes client-side
  - Click recipe card → navigate to `/recipe/{id}`
- **Animation**: Non-matching cards fade out with stagger effect

**4. Recipe Detail Screen** (`/recipe/{id}`)
- **Components**:
  - `RecipeHeader` - Hero image, title, metadata
  - `ServingSizeSelector` - Dropdown or stepper (1-6 servings)
  - `EquipmentList` - Required tools
  - `IngredientList` - Categorized ingredients
  - `InstructionSteps` - Mise en place, cooking, plating sections
  - `SendToPhoneButton` - Shopping list generation
  - `SubstitutionNotification` - Real-time sync indicator
- **Data Flow**:
  1. Initial load: Fetch recipe from Supabase (no amounts shown)
  2. User selects serving size → API call to `/api/calculate-recipe`
  3. Server returns scaled amounts → populate UI
  4. Real-time subscription listens for shopping list updates
- **State Management**:
  - Selected serving size (local state)
  - Calculated recipe data (local state)
  - Substitutions made (synced from Supabase)

**5. API Routes**

**`/api/calculate-recipe`** (POST)
- **Input**: `{ recipeId, servings }`
- **Process**:
  - Fetch base recipe (default servings: 2)
  - Calculate multiplier: `servings / baseServings`
  - Scale all ingredient amounts
  - Scale all instruction quantities
  - Return scaled recipe
- **Output**: Complete recipe with all amounts calculated

**`/api/shopping-list/create`** (POST)
- **Input**: `{ userId, recipeId, servings, calculatedIngredients }`
- **Process**:
  - Create shopping list record in Supabase
  - Create shopping list items (one per ingredient)
  - Organize by category (Proteins, Produce, Pantry, Dairy)
  - Trigger push notification to iOS app
- **Output**: `{ shoppingListId, status: 'sent' }`

---

### iOS Application (SwiftUI)

The iOS app is Isabelle's shopping companion at the store.

#### Core Screens

**1. Shopping List View** (`ContentView`)
- **Components**:
  - `ShoppingListHeader` - Recipe name, serving size
  - `CategorySection` - Grouped by category
  - `ShoppingListItem` - Checkbox, ingredient name, amount
- **Data**: Fetched from Supabase on push notification
- **Interaction**:
  - Tap checkbox → mark as purchased
  - Tap item → navigate to `ItemDetailView`
- **Offline Support**: CoreData cached copy of list

**2. Item Detail View** (`ItemDetailView`)
- **Components**:
  - `ItemHeader` - Ingredient name and amount
  - `RecommendedBrands` - Pre-programmed brand list
  - `AlternativeBrands` - Trusted substitutes
  - `SubstitutionOptions` - Alternative ingredients with conversion
- **Data**:
  - Static substitution database (bundled with app)
  - Organized by ingredient type
- **Interaction**:
  - Tap "Use Thai Kitchen" → update item in Supabase
  - Mark item as checked → sync back

**3. Real-time Sync Service**
- **Technology**: Supabase Realtime subscriptions
- **Process**:
  - Subscribe to `shopping_list_items` table changes
  - Listen for updates where `user_id = currentUser`
  - Update local CoreData cache
  - Post notification to update UI

---

## Data Model

### Supabase Database Schema

#### Core Tables

**`users`**
```sql
id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4()
email               TEXT UNIQUE NOT NULL
created_at          TIMESTAMP WITH TIME ZONE DEFAULT NOW()
phone_device_token  TEXT  -- For push notifications
```

**`proteins`**
```sql
id          UUID PRIMARY KEY DEFAULT uuid_generate_v4()
name        TEXT NOT NULL  -- "Shrimp", "Chicken", etc.
slug        TEXT UNIQUE NOT NULL  -- "shrimp", "chicken"
image_url   TEXT NOT NULL
sort_order  INTEGER
```

**`cuisines`**
```sql
id          UUID PRIMARY KEY DEFAULT uuid_generate_v4()
name        TEXT NOT NULL  -- "Thai", "Italian", etc.
slug        TEXT UNIQUE NOT NULL  -- "thai", "italian"
image_url   TEXT NOT NULL
description TEXT
sort_order  INTEGER
```

**`recipes`**
```sql
id               UUID PRIMARY KEY DEFAULT uuid_generate_v4()
name             TEXT NOT NULL
slug             TEXT UNIQUE NOT NULL
description      TEXT
protein_id       UUID REFERENCES proteins(id)
cuisine_id       UUID REFERENCES cuisines(id)
image_url        TEXT NOT NULL
prep_time_min    INTEGER
cook_time_min    INTEGER
total_time_min   INTEGER
difficulty       TEXT CHECK (difficulty IN ('Easy', 'Medium', 'Hard'))
default_servings INTEGER DEFAULT 2
created_at       TIMESTAMP WITH TIME ZONE DEFAULT NOW()
```

**`recipe_equipment`**
```sql
id          UUID PRIMARY KEY DEFAULT uuid_generate_v4()
recipe_id   UUID REFERENCES recipes(id) ON DELETE CASCADE
equipment   TEXT NOT NULL  -- "Large wok or deep skillet"
category    TEXT  -- "Cooking", "Prep", "Measuring"
sort_order  INTEGER
```

**`recipe_ingredients`**
```sql
id            UUID PRIMARY KEY DEFAULT uuid_generate_v4()
recipe_id     UUID REFERENCES recipes(id) ON DELETE CASCADE
ingredient    TEXT NOT NULL  -- "large shrimp"
amount        DECIMAL  -- 1
unit          TEXT  -- "lb" or "lbs"
category      TEXT  -- "Proteins", "Produce", "Pantry", "Dairy"
is_secondary  BOOLEAN DEFAULT FALSE  -- For ingredient wheel filtering
notes         TEXT  -- "peeled and deveined"
sort_order    INTEGER
```

**`recipe_instructions`**
```sql
id                 UUID PRIMARY KEY DEFAULT uuid_generate_v4()
recipe_id          UUID REFERENCES recipes(id) ON DELETE CASCADE
phase              TEXT CHECK (phase IN ('mise_en_place', 'cooking', 'plating'))
step_number        INTEGER
instruction        TEXT NOT NULL
has_quantity       BOOLEAN DEFAULT FALSE  -- Does this step include amounts?
clean_as_you_go    TEXT  -- Optional cleanup note
```

**`secondary_ingredients`**
```sql
id          UUID PRIMARY KEY DEFAULT uuid_generate_v4()
recipe_id   UUID REFERENCES recipes(id) ON DELETE CASCADE
ingredient  TEXT NOT NULL  -- "Coconut", "Lime", "Basil"
slug        TEXT NOT NULL  -- "coconut", "lime", "basil"
```
*Used to populate the ingredient wheel for filtering*

**`shopping_lists`**
```sql
id          UUID PRIMARY KEY DEFAULT uuid_generate_v4()
user_id     UUID REFERENCES users(id) ON DELETE CASCADE
recipe_id   UUID REFERENCES recipes(id)
servings    INTEGER NOT NULL
created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW()
status      TEXT DEFAULT 'active' CHECK (status IN ('active', 'completed'))
```

**`shopping_list_items`**
```sql
id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4()
shopping_list_id    UUID REFERENCES shopping_lists(id) ON DELETE CASCADE
ingredient          TEXT NOT NULL
amount              DECIMAL
unit                TEXT
category            TEXT
is_checked          BOOLEAN DEFAULT FALSE
recommended_brand   TEXT
substitution_used   TEXT  -- NULL or name of substitute used
notes               TEXT
sort_order          INTEGER
```

**`ingredient_substitutions`**
```sql
id                    UUID PRIMARY KEY DEFAULT uuid_generate_v4()
ingredient            TEXT NOT NULL
recommended_brands    JSONB  -- ["Mae Ploy", "Aroy-D"]
alternative_brands    JSONB  -- ["Thai Kitchen"]
substitutes           JSONB  -- [{ "name": "Yellow curry paste", ... }]
```
*Pre-programmed substitution knowledge (no AI)*

---

## User Flow: Technical Implementation

### Mapping Isabelle's Journey to Components

#### Scene 1-3: Discovery Flow

**User Action**: Browse proteins → cuisines → recipes

**Technical Flow**:
1. Navigate to `/` → `ProteinGrid` renders static data
2. Click "Shrimp" → Navigate to `/cuisine?protein=shrimp`
3. `CuisineGrid` filters cuisines compatible with shrimp (could be static or DB query)
4. Click "Thai" → Navigate to `/recipes?protein=shrimp&cuisine=thai`
5. Client fetches recipes:
```typescript
const { data: recipes } = await supabase
  .from('recipes')
  .select(`
    *,
    proteins(name, slug),
    cuisines(name, slug)
  `)
  .eq('proteins.slug', 'shrimp')
  .eq('cuisines.slug', 'thai')
```
6. Generate ingredient wheel from `secondary_ingredients`:
```typescript
const { data: secondaryIngredients } = await supabase
  .from('secondary_ingredients')
  .select('ingredient, slug')
  .in('recipe_id', recipeIds)
  .distinct()
```

#### Scene 4: Recipe Detail & Serving Selection

**User Action**: View recipe, select 4 servings, see amounts populate

**Technical Flow**:
1. Navigate to `/recipe/green-curry-shrimp`
2. Fetch base recipe (no amounts shown in UI yet):
```typescript
const { data: recipe } = await supabase
  .from('recipes')
  .select(`
    *,
    recipe_equipment(*),
    recipe_ingredients(*),
    recipe_instructions(*)
  `)
  .eq('slug', 'green-curry-shrimp')
  .single()
```
3. User selects 4 servings from dropdown
4. Call API to calculate:
```typescript
const response = await fetch('/api/calculate-recipe', {
  method: 'POST',
  body: JSON.stringify({ recipeId, servings: 4 })
})
const scaledRecipe = await response.json()
```
5. Server calculation logic:
```typescript
const multiplier = servings / recipe.default_servings  // 4 / 2 = 2
const scaledIngredients = recipe.recipe_ingredients.map(ing => ({
  ...ing,
  amount: ing.amount * multiplier
}))
```
6. Update UI with amounts instantly

#### Scene 5: Send to iPhone

**User Action**: Click "Send Shopping List to iPhone"

**Technical Flow**:
1. Call API:
```typescript
const response = await fetch('/api/shopping-list/create', {
  method: 'POST',
  body: JSON.stringify({
    userId,
    recipeId,
    servings: 4,
    ingredients: scaledRecipe.ingredients
  })
})
```
2. Server creates records:
```typescript
// Create shopping list
const { data: shoppingList } = await supabase
  .from('shopping_lists')
  .insert({ user_id: userId, recipe_id: recipeId, servings })
  .select()
  .single()

// Create items
const items = ingredients.map(ing => ({
  shopping_list_id: shoppingList.id,
  ingredient: ing.ingredient,
  amount: ing.amount,
  unit: ing.unit,
  category: ing.category,
  recommended_brand: getRecommendedBrand(ing.ingredient)
}))

await supabase.from('shopping_list_items').insert(items)
```
3. Send push notification to iOS:
```typescript
await sendPushNotification({
  deviceToken: user.phone_device_token,
  title: 'Shopping List Ready',
  body: `${recipe.name} (${servings} servings)`,
  data: { shoppingListId: shoppingList.id }
})
```

#### Scene 6: Shopping & Substitution

**User Action**: In store, can't find Mae Ploy curry paste, selects Thai Kitchen

**Technical Flow** (iOS):
1. Tap on "Gluten-free green curry paste" item
2. Navigate to `ItemDetailView`
3. Fetch substitution data (from local bundled database):
```swift
let substitutions = SubstitutionDatabase.shared
  .getSubstitutions(for: "green curry paste")
```
4. Display recommended brands and alternatives
5. User taps "Use Thai Kitchen Green Curry Paste"
6. Update Supabase:
```swift
try await supabase
  .from("shopping_list_items")
  .update([
    "is_checked": true,
    "substitution_used": "Thai Kitchen green curry paste"
  ])
  .eq("id", item.id)
  .execute()
```
7. Real-time trigger fires → web app receives update

#### Scene 7: Cooking with Updated Recipe

**User Action**: Opens laptop, sees substitution notification

**Technical Flow** (Web):
1. Recipe detail screen has active Realtime subscription:
```typescript
const subscription = supabase
  .channel('shopping-list-updates')
  .on('postgres_changes', {
    event: 'UPDATE',
    schema: 'public',
    table: 'shopping_list_items',
    filter: `shopping_list_id=eq.${shoppingListId}`
  }, (payload) => {
    // Update UI with substitution
    setSubstitutions(prev => [...prev, payload.new])
  })
  .subscribe()
```
2. UI shows notification badge: "Shopping list updated - View substitutions"
3. Display substitution panel:
```
Substitutions Made:
- ~~Gluten-free green curry paste (Mae Ploy)~~
- ✓ Thai Kitchen green curry paste (same quantity)
```

---

## Real-Time Synchronization

### How iOS ↔ Web Sync Works

**Architecture**:
```text
┌─────────────────────────────────────────────────────┐
│                   Supabase Realtime                 │
│                   (WebSocket Server)                │
└──────────────┬───────────────────────┬──────────────┘
               │                       │
        [Subscribe]              [Subscribe]
               │                       │
         ┌─────┴──────┐         ┌─────┴──────┐
         │  Web App   │         │  iOS App   │
         │            │         │            │
         │  Listens:  │         │  Listens:  │
         │  Changes   │         │  Changes   │
         └─────┬──────┘         └─────┬──────┘
               │                       │
          [Receives]               [Updates]
               │                       │
               └───────────┬───────────┘
                           │
                    shopping_list_items
```

**Web App Subscription** (React hook):
```typescript
useEffect(() => {
  if (!shoppingListId) return

  const channel = supabase
    .channel(`shopping-list-${shoppingListId}`)
    .on('postgres_changes', {
      event: 'UPDATE',
      schema: 'public',
      table: 'shopping_list_items',
      filter: `shopping_list_id=eq.${shoppingListId}`
    }, (payload) => {
      // Payload contains old and new row data
      handleSubstitutionUpdate(payload.new)
    })
    .subscribe()

  return () => {
    channel.unsubscribe()
  }
}, [shoppingListId])
```

**iOS App Subscription** (Swift):
```swift
let channel = supabase.channel("shopping-list-\(shoppingListId)")

channel.on(.postgresChanges(
  event: .update,
  schema: "public",
  table: "shopping_list_items",
  filter: "shopping_list_id=eq.\(shoppingListId)"
)) { payload in
  // Update local CoreData cache
  updateLocalItem(with: payload.new)
}

channel.subscribe()
```

---

## Key Technical Decisions

### 1. Static vs. Dynamic Data

**Static** (Bundled with app):
- Protein images and metadata
- Cuisine images and metadata
- Substitution knowledge base (iOS app)

**Dynamic** (From Supabase):
- Recipes and all recipe data
- User shopping lists
- Shopping list items and substitutions
- User preferences and history (future)

**Rationale**:
- Static data reduces API calls and improves performance
- Dynamic recipe data allows easy updates without app releases
- Substitution data is bundled in iOS for offline shopping

### 2. Serving Size Calculation

**Server-side** (not client-side)

**Why**:
- Single source of truth for scaling logic
- Can handle complex conversions (e.g., 1.5 cups → 1 cup + 2 tbsp)
- Easier to test and maintain
- Can add unit conversions later (metric ↔ imperial)

### 3. Real-time vs. Polling

**Real-time subscriptions** (Supabase Realtime)

**Why**:
- Instant updates (Isabelle sees substitution immediately)
- More efficient than polling
- Better user experience
- Built into Supabase (no extra infrastructure)

### 4. Ingredient Wheel Generation

**Dynamic** (generated from matching recipes)

**Why**:
- Always accurate to available recipes
- Automatically updates when recipes added/removed
- No manual curation needed
- Shows only relevant filters for current selection

---

## Future Enhancements & Extensibility

### Phase 2 Features (Post-Alpha)

**Meal Planning Calendar**:
- New table: `meal_plans` (user_id, date, recipe_id)
- New screen: Weekly calendar view
- Batch shopping list generation

**Recipe Collections**:
- New table: `recipe_collections` (user_id, name, recipe_ids[])
- User-curated favorites and themed collections

**Pantry Tracking**:
- New table: `pantry_items` (user_id, ingredient, amount, expiration)
- Smart shopping lists (exclude items in pantry)

**Nutritional Information**:
- Add columns to `recipe_ingredients`: calories, protein, carbs, fat
- Aggregate and display per serving

**Social Features**:
- New table: `recipe_reviews` (user_id, recipe_id, rating, comment)
- New table: `user_follows` (follower_id, following_id)
- Share recipes, photos, and reviews

---

## Success Criteria

### Performance Targets

- **Initial page load**: < 2 seconds
- **Protein → Cuisine → Recipes navigation**: < 500ms per transition
- **Serving size calculation**: < 1 second
- **Real-time sync latency**: < 2 seconds (iOS → Web)
- **Shopping list generation**: < 3 seconds

### User Experience Metrics

- **Planning time**: < 10 minutes (select recipe + send to phone)
- **Shopping confidence**: 100% (substitution help always available)
- **Kitchen stress**: Minimal (clean-as-you-go, mise en place structure)
- **Meal quality**: Restaurant-level (fresh ingredients, clear instructions)

---

## Implementation Phases

### MVP (Minimum Viable Product)

**Scope**: Core flow from Isabelle's story (Scenes 1-7)

**Features**:
- ✅ Protein/cuisine/recipe discovery
- ✅ Ingredient wheel filtering
- ✅ Serving size selection and calculation
- ✅ Shopping list generation and sync
- ✅ Substitution help and tracking
- ✅ Real-time iOS ↔ Web sync

**Timeline**: 6-8 weeks

### Alpha Release

**Scope**: MVP + polish + 20-30 recipes

**Features**:
- MVP features (above)
- Responsive design (mobile web support)
- Error handling and loading states
- User authentication
- Recipe search (simple text search)

**Timeline**: +2 weeks

### Beta Release

**Scope**: Alpha + expanded recipe library + analytics

**Features**:
- 100+ recipes across multiple cuisines
- Recipe ratings and reviews
- User favorites/bookmarks
- Usage analytics
- Performance optimizations

**Timeline**: +4 weeks

---

## Conclusion

This system design creates a joyful, confident cooking experience for people with celiac disease. Every component - from visual protein selection to real-time substitution sync - is purpose-built to eliminate anxiety and empower creativity.

The technical architecture is deliberately simple:
- **One database** (Supabase PostgreSQL)
- **One web framework** (Next.js)
- **One mobile platform** (iOS/SwiftUI)
- **One sync mechanism** (Supabase Realtime)

This simplicity ensures rapid development, easy maintenance, and a smooth migration from local development to production.
