# 🍳 Forkify — Recipe Search & Bookmark Application

A recipe search application built while following **Jonas Schmedtmann's** [JavaScript course](https://www.udemy.com/course/the-complete-javascript-course/). Forkify allows users to search for recipes, view detailed recipe information, adjust serving sizes with dynamic ingredient recalculation, bookmark favorites, and upload their own recipes.

![Forkify](src/img/logo.png)

---

## 🚀 Quick Start

```bash
# Install dependencies
npm install

# Run the development server
npm start

# Build for production
npm run build
```

---

## 📸 Features

| Feature                    | Description                                                                                                                         |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Recipe Search**          | Search thousands of recipes via the Forkify API. Results are paginated at 10 per page.                                              |
| **Recipe Detail View**     | View recipe image, cooking time, servings, ingredients (with fractional display), source link, and publisher info.                  |
| **Servings Adjustment**    | Increase or decrease servings — ingredient quantities recalculate proportionally in real time.                                      |
| **Bookmarking**            | Save favorite recipes. Bookmarks persist across page reloads via `localStorage`.                                                    |
| **Add Recipe**             | Upload custom recipes through a modal form. Ingredients are parsed from a `Quantity,Unit,Description` format and POSTed to the API. |
| **Hash-based Routing**     | Recipe IDs are stored in the URL hash (`#recipeId`). Navigating or refreshing restores the current recipe.                          |
| **Spinner & Error States** | Animated loading spinner during async operations; user-friendly error and success messages.                                         |
| **DOM Diffing**            | Efficient UI updates — only changed text and attributes are patched, avoiding full re-renders.                                      |

---

## 🧠 What I Learned

This project was a turning point in my understanding of JavaScript. Below is a breakdown of the key concepts I internalized by building Forkify step by step.

### 1. MVC Architecture

The single most impactful lesson was learning how to **separate concerns** using the Model-View-Controller pattern:

- **Model** — Holds all state and business logic. No DOM knowledge. Pure data.
- **View** — Renders the DOM and captures user events. No business logic. Just presentation.
- **Controller** — The orchestrator that wires everything together. It's the only layer that knows about both the model and the views.

This pattern made the code predictable, testable, and easy to extend. I now understand why mixing data logic and UI code in one file leads to spaghetti code.

### 2. Asynchronous JavaScript

Before this project, `async/await` felt abstract. Building Forkify forced me to use it in a real context:

- Every network call (`loadRecipe`, `loadSearchResults`, `uploadRecipe`) uses `async/await`.
- I learned how `Promise.race` can implement request timeouts — racing the fetch against a timer promise.
- I practiced proper error propagation: catching API errors, re-throwing with meaningful messages, and displaying them in the UI.

### 3. ES6 Modules

The entire app is built with `import`/`export`. I learned:

- **Named exports** for config values and helper functions.
- **Default exports** for view singletons (`export default new RecipeView()`).
- **Namespace imports** (`import * as model from './model.js'`) to group related exports under one object.
- How Parcel resolves and bundles the module graph — no manual config needed.

### 4. Object-Oriented Programming with Classes

I built a **class hierarchy** for the views:

- `View` is an abstract-style base class with shared methods (`render`, `update`, `renderSpinner`, `renderError`, `renderMessage`).
- Each concrete view (`RecipeView`, `SearchView`, etc.) extends `View` and implements `_generateMarkup()`.
- The `update()` method implements a lightweight **virtual DOM diffing** algorithm — comparing current and new DOM nodes with `isEqualNode()` and patching only what changed.

This taught me inheritance, method overriding, and why protected-style naming (`_` prefix) signals internal APIs.

### 5. Event Delegation & Closures

Instead of attaching listeners to every button, I used **event delegation** on parent elements with `e.target.closest()`:

- Pagination buttons, servings buttons, bookmark toggles — all handled via delegation on their parent container.
- Handler functions from the controller are passed into views via `addHandler*` methods, creating **closures** that give views access to model operations without importing the model directly.

### 6. State Management & Persistence

The `model.state` object is the single source of truth for the entire app:

- `state.recipe` — currently viewed recipe.
- `state.search` — search results, query, and current page.
- `state.bookmarks` — array of bookmarked recipes, persisted to `localStorage` and restored on app init.

I learned why centralizing state makes debugging easier and prevents inconsistent UI.

### 7. API Integration (REST: GET & POST)

I integrated a real REST API from scratch:

- **GET requests** — fetching recipes and search results using `fetch()`.
- **POST requests** — uploading new recipes with `JSON.stringify` payload.
- **Response validation** — checking `res.ok` and throwing descriptive errors.
- **Timeout handling** — wrapping fetches in `Promise.race` to reject after 10 seconds.

### 8. Modern DOM APIs

Several DOM APIs I hadn't used before became essential:

- `FormData` + `Object.fromEntries()` — collecting and converting form input into a plain object.
- `document.createRange().createContextualFragment()` — parsing HTML strings into DOM fragments for diffing.
- `Node.isEqualNode()` — comparing virtual and real DOM nodes.
- `window.history.pushState` — updating the URL without a page reload.
- `insertAdjacentHTML` — rendering markup efficiently without innerHTML.

### 9. SASS & CSS Architecture

I practiced professional CSS methodologies:

- **BEM naming** — Block-Element-Modifier for all class names (e.g., `recipe__info-data--minutes`).
- **7-1 partial structure** — one main file importing seven partials.
- **SCSS variables** — centralized color palette, breakpoints, and gradients.
- **`@extend` with placeholder selectors** — `%btn` shared across button variants.
- **CSS Grid + Flexbox** — Grid for the overall layout and ingredient lists; Flexbox for component-level alignment.
- **Responsive design** — four breakpoints using rem-based font sizing (62.5% root).

### 10. Build Tooling with Parcel

Parcel 2 was my first zero-config bundler experience:

- It automatically detects the entry point from `index.html`.
- It processes SASS via `@parcel/transformer-sass`.
- It handles `url:` prefixed imports for static assets (`import icons from 'url:../../img/icons.svg'`).
- `core-js` and `regenerator-runtime` provide polyfills for older browsers.

I now understand why bundlers exist and how they transform modular source code into deployable bundles.

---

## 📁 Project Structure

```
Forkify/
├── index.html                  # Entry point — all static HTML markup
├── package.json                # Dependencies, scripts, metadata
├── .prettierrc                 # Prettier config (singleQuote, arrowParens)
│
└── src/
    ├── img/
    │   ├── favicon.png
    │   ├── icons.svg           # SVG sprite (symbol-based icon system)
    │   └── logo.png
    │
    ├── js/
    │   ├── config.js           # Centralized constants (API_URL, TIMEOUT_SEC, KEY, etc.)
    │   ├── helpers.js          # AJAX helper (GET/POST), timeout promise
    │   ├── model.js            # State object + all business logic
    │   ├── controller.js       # Orchestrator — wires model and views
    │   └── views/
    │       ├── View.js             # Abstract parent class
    │       ├── recipeView.js       # Recipe detail rendering
    │       ├── searchView.js       # Search form handling
    │       ├── resultsView.js      # Search results list
    │       ├── previewView.js      # Individual result/bookmark card
    │       ├── paginationView.js   # Prev/next page controls
    │       ├── bookmarksView.js    # Bookmarks dropdown panel
    │       └── addRecipeView.js    # Upload recipe modal form
    │
    └── sass/
        ├── main.scss           # Imports all partials
        ├── _base.scss          # Reset, variables, global styles, grid
        ├── _components.scss    # Buttons, spinner, error/message, headings
        ├── _header.scss        # Header, search bar, nav, bookmarks
        ├── _searchResults.scss # Results list, pagination, copyright
        ├── _recipe.scss        # Recipe figure, details, ingredients
        ├── _preview.scss       # Search result/bookmark preview cards
        └── _upload.scss        # Modal, overlay, upload form
```

---

## 🛠 Tech Stack

| Category         | Technology                              |
| ---------------- | --------------------------------------- |
| **Language**     | JavaScript (ES6+), SASS                 |
| **Bundler**      | Parcel 2                                |
| **Architecture** | MVC                                     |
| **Styling**      | BEM, SCSS, CSS Grid, Flexbox            |
| **API**          | Forkify API (REST — GET & POST)         |
| **Polyfills**    | core-js, regenerator-runtime            |
| **Utilities**    | fractional (decimal → fraction display) |
| **Formatting**   | Prettier                                |

---

## 📄 License

This project is licensed under **Creative Commons Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)**. See the [LICENSE](LICENSE) file for full details.

**Attribution**: This project was built while following Jonas Schmedtmann's JavaScript course. The design and original code belong to Jonas Schmedtmann and Omnifood, Inc. I built this implementation myself by following the course and understanding the concepts applied.

---

> _Built as a learning project while following Jonas Schmedtmann's JavaScript course. All design and instructional content belong to Jonas Schmedtmann._
