[![npm version](https://img.shields.io/npm/v/@itrocks/breadcrumb?logo=npm)](https://www.npmjs.org/package/@itrocks/breadcrumb)
[![npm downloads](https://img.shields.io/npm/dm/@itrocks/breadcrumb)](https://www.npmjs.org/package/@itrocks/breadcrumb)
[![GitHub](https://img.shields.io/github/last-commit/itrocks-ts/breadcrumb?color=2dba4e&label=commit&logo=github)](https://github.com/itrocks-ts/breadcrumb)
[![issues](https://img.shields.io/github/issues/itrocks-ts/breadcrumb)](https://github.com/itrocks-ts/breadcrumb/issues)
[![discord](https://img.shields.io/discord/1314141024020467782?color=7289da&label=discord&logo=discord&logoColor=white)](https://25.re/ditr)

# breadcrumb

Dynamically updates the breadcrumb based on containers with data-action 'list' or 'view'.

*This documentation was written by an artificial intelligence and may contain errors or approximations.
It has not yet been fully reviewed by a human. If anything seems unclear or incomplete,
please feel free to contact the author of this package.*

## Installation

```bash
npm i @itrocks/breadcrumb
```

## Usage

`@itrocks/breadcrumb` is a tiny browser helper that keeps an HTML
breadcrumb in sync with the current *list* / *view* page displayed in a
`<main>` area.

It expects:

- a single `<ol class="breadcrumb">` element somewhere in the
  `document.body`, and
- page content rendered inside `<main>` as elements with a
  `data-action` attribute equal to either `list` or `view`.

You call the exported `breadcrumb` function with the heading element of
the current page; it will create or update the corresponding
`<li>` in the breadcrumb and link it to the current URL.

### Minimal example

```html
<body>
  <ol class="breadcrumb"></ol>

  <main>
    <article data-action="list">
      <h1 id="customers-heading">Customers</h1>
      <!-- list content -->
    </article>
  </main>

  <script type="module">
    import { breadcrumb } from '@itrocks/breadcrumb'

    const heading = document.getElementById('customers-heading') as HTMLHeadingElement
    breadcrumb(heading)
  </script>
</body>
```

After calling `breadcrumb(heading)`, the `<ol class="breadcrumb">`
contains (simplified):

```html
<ol class="breadcrumb">
  <li data-action="list">
    <a href="/customers" target="main">Customers</a>
  </li>
</ol>
```

### Complete example with list / view navigation

In a typical it.rocks layout, list and view pages are loaded into a
`<main>` frame, often via links with `target="main"`. You can update the
breadcrumb whenever a page is loaded by calling `breadcrumb` on the
page's main heading.

```html
<body>
  <!-- Shared layout -->
  <ol class="breadcrumb"></ol>

  <main name="main">
    <!-- Example list page -->
    <article data-action="list" id="users-list">
      <h1 id="users-heading">Users</h1>
      <ul>
        <li><a href="/users/1" target="main">Alice</a></li>
        <li><a href="/users/2" target="main">Bob</a></li>
      </ul>
    </article>
  </main>

  <script type="module">
    import { breadcrumb } from '@itrocks/breadcrumb'

    function updateBreadcrumbForCurrentPage () {
      const heading = document.querySelector('main > [data-action] h1') as HTMLHeadingElement | null
      if (heading) breadcrumb(heading)
    }

    // Initial load
    updateBreadcrumbForCurrentPage()

    // If you use client-side navigation, call updateBreadcrumbForCurrentPage()
    // again each time you replace the content of <main>.
  </script>
</body>
```

When a *view* page is displayed instead of the list, for example:

```html
<main name="main">
  <article data-action="view">
    <h1 id="user-heading">Alice</h1>
    <!-- user details -->
  </article>
</main>

<script type="module">
  import { breadcrumb } from '@itrocks/breadcrumb'

  const heading = document.getElementById('user-heading') as HTMLHeadingElement
  breadcrumb(heading)
</script>
```

the breadcrumb will contain both levels (list and view):

```html
<ol class="breadcrumb">
  <li data-action="list">
    <a href="/users" target="main">Users</a>
  </li>
  <li data-action="view">
    <a href="/users/1" target="main">Alice</a>
  </li>
</ol>
```

If you navigate back to the list, the `data-action="view"` item is
removed automatically.

## API

### `function breadcrumb(heading: HTMLHeadingElement): void`

Updates the `<ol class="breadcrumb">` element to reflect the current
page represented by `heading`.

#### Parameters

- `heading: HTMLHeadingElement` – The main heading of the current page.
  This heading **must** be inside an element that:
  - is a direct child of `<main>` (`main > [data-action]`), and
  - has a `data-action` attribute whose value indicates the kind of
    page:
    - `'list'` – a listing page (master list of items),
    - any other value (typically `'view'`) – a detail page for a single
      item.

#### Behaviour

When called, `breadcrumb`:

1. Locates the closest ancestor of `heading` that matches
   `main > [data-action]`. If none is found, nothing happens.
2. Locates a single `<ol class="breadcrumb">` in `document.body`. If not
   found, nothing happens.
3. Creates an `<a>` element using:
   - `href = window.location.href` (current URL),
   - `target = 'main'` (so that following the link reloads into the
     `<main>` frame),
   - `textContent = heading.textContent`.
4. Determines the current action:
   - `'list'` if `data-action="list"` on the container,
   - `'view'` for any other value.
5. If the current action is `'list'`, removes any existing
   `<li data-action="view">` entry from the breadcrumb.
6. Finds or creates a `<li data-action="list|view">` corresponding to
   the current action, clears its content, appends the new `<a>`, and
   ensures it is appended to the breadcrumb list.

This function is idempotent for the same page URL and heading: calling
it multiple times simply refreshes the corresponding breadcrumb item.

## Typical use cases

- **Master/detail navigation** – Represent the path `List → Detail`
  (for example `Users → Alice`) in the breadcrumb while pages are loaded
  into a `<main>` frame.
- **Frame-based layouts** – Combine with links using `target="main"`
  so that breadcrumb links reload the correct content inside the
  application frame instead of performing a full page refresh.
- **Lightweight breadcrumb in server-rendered apps** – Add dynamic
  breadcrumb behaviour without introducing a full client-side router;
  simply call `breadcrumb(heading)` after each page load.
- **Integration with it.rocks actions** – Use together with pages
  generated by `@itrocks/list`, `@itrocks/view`, or equivalent actions
  that render `main > [data-action]` containers, so navigation is
  reflected in the breadcrumb automatically.
