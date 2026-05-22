# Storybook

---

## Overview

Add Storybook to the project to develop, document, and showcase UI components
in isolation. This makes it easier to build and review components independently
of the full application, and provides a living style guide for the design system.

> **Note:** This spec covers React, Svelte, and plain JavaScript as worked examples.
> Storybook supports other frameworks too — and a handcrafted Storybook instance
> can be configured to suit any project's specific needs. See the
> [Storybook documentation](https://storybook.js.org/docs) for the full list of
> supported frameworks and configuration options.

---

## Key

A reference for the prefixes used throughout this document.

| Prefix | Stands for | Description |
|--------|------------|-------------|
| `US-##` | User Story | A feature requirement from the user's perspective |
| `AC-##` | Acceptance Criteria | A condition that must be met for the story to be complete |

---

## User stories

Each user story describes a requirement from the user's perspective, followed by
the acceptance criteria that must be met for it to be considered complete.

### US-01 — Component development in isolation

> As a developer, I want to build and view components in isolation so I can
> develop and test them without needing the full application running.

- [ ] **AC-01** — Storybook runs locally via `npm run storybook`
- [ ] **AC-02** — Each component has at least one story showing its default state
- [ ] **AC-03** — Component props / attributes can be adjusted interactively in the Storybook UI

### US-02 — Component documentation

> As a developer or designer, I want to see documentation for each component
> alongside its visual representation so I can understand how to use it.

- [ ] **AC-01** — Each story includes a description of the component and its variants
- [ ] **AC-02** — Props / attributes are documented in the Storybook controls panel
- [ ] **AC-03** — Usage examples are visible alongside the rendered component

### US-03 — Design system showcase

> As a stakeholder, I want to browse all components and patterns in one place
> so I can review the design system without needing access to the codebase.

- [ ] **AC-01** — Storybook can be built as a static site via `npm run build-storybook`
- [ ] **AC-02** — The static build can be deployed independently of the main project
- [ ] **AC-03** — All components are browsable and searchable in the built output

---

## Out of scope

Explicitly listing what this feature does NOT cover prevents scope creep and
helps the agent avoid building more than is required.

- Visual regression testing (e.g. Chromatic)
- Accessibility testing addons
- Automated story generation
- Integration with a CMS or design tool

---

## Setup guidance

The following covers React, Svelte, and plain JavaScript as worked examples. For Astro,
Eleventy, or any other framework, refer to the
[Storybook documentation](https://storybook.js.org/docs) for setup instructions.

### React

```bash
npx storybook@latest init
```

Storybook detects React automatically. Stories use `.stories.jsx` or `.stories.tsx`.

```jsx
// src/components/Button/Button.stories.jsx
import Button from './Button';

export default {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
};

export const Primary = {
  args: { label: 'Click me', variant: 'primary' },
};
```

### Svelte

```bash
npx storybook@latest init
```

Storybook detects Svelte automatically. Stories use `.stories.js` or `.stories.svelte`.

```js
// src/components/Button/Button.stories.js
import Button from './Button.svelte';

export default {
  title: 'Components/Button',
  component: Button,
  tags: ['autodocs'],
};

export const Primary = {
  args: { label: 'Click me', variant: 'primary' },
};
```

### Plain JavaScript

```bash
npx storybook@latest init --type html
```

Stories use `.stories.js` and use a `render` function to map args to an HTML string or
DOM element. This is how interactive controls work in the `@storybook/html` format.

```js
// src/components/Button/Button.stories.js
export default {
  title: 'Components/Button',
  tags: ['autodocs'],
  argTypes: {
    label: { control: 'text' },
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'danger', 'ghost'],
    },
  },
};

export const Primary = {
  args: { label: 'Click me', variant: 'primary' },
  render: (args) => `
    <button class="btn btn--${args.variant}">${args.label}</button>
  `,
};
```

---

## Storybook configuration

Storybook's config lives in a `.storybook/` directory at the project root. The two
files that always need attention are `main.js` and `preview.js`. The `init` command
generates both — the notes below cover what to add to each after setup.

### `main.js`

Controls which files Storybook treats as stories. Verify the `stories` glob matches
the project's file layout. For this scaffold it should be:

```js
stories: ['../src/**/*.stories.{js,jsx,ts,tsx,svelte}'],
```

### `preview.js`

Runs before every story. This is where global styles and decorators are registered.

**Import global styles** so design tokens are available inside the Storybook canvas:

```js
// .storybook/preview.js
import '../src/styles/main.css';
```

Without this, every story renders with unstyled components because the Storybook
canvas is a separate iframe with its own `<html>` document — it does not inherit
styles from the main application.

**Add a theme decorator** so the `data-theme` attribute is applied to the story
root, matching the mechanism used by the `ThemeToggle` component in the main app:

```js
// .storybook/preview.js
import '../src/styles/main.css';

export const decorators = [
  (Story, context) => {
    const theme = context.globals.theme ?? 'light';
    const wrapper = document.createElement('div');
    wrapper.setAttribute('data-theme', theme);
    const story = Story();
    wrapper.appendChild(typeof story === 'string'
      ? Object.assign(document.createElement('div'), { innerHTML: story })
      : story
    );
    return wrapper;
  },
];

export const globalTypes = {
  theme: {
    name: 'Theme',
    defaultValue: 'light',
    toolbar: {
      icon: 'circlehollow',
      items: ['light', 'dark'],
      showName: true,
    },
  },
};
```

This adds a Theme switcher to the Storybook toolbar so stories can be previewed in
both light and dark mode. The decorator applies `data-theme` to a wrapper element
rather than directly to `document.documentElement` to keep stories isolated from
each other.

> **Note:** The decorator above is written for the `@storybook/html` (Vanilla) format.
> For React and Svelte, the `Story` return value is a component, not a string — wrap
> it in a container element with `data-theme` set instead. The `globalTypes` block is
> identical across all three.

---

## Notes for AI agents

- Run `npx storybook@latest init` and let Storybook auto-detect the active framework
- If auto-detection fails, pass the `--type` flag manually based on the active framework
  selection in `docs/project-brief.md`:

  | Framework (project-brief.md) | `--type` value |
  |------------------------------|----------------|
  | Vanilla | `html` |
  | React | `react` |
  | Svelte | `svelte` |
  | Astro | see Storybook docs — do not guess |
  | Eleventy | see Storybook docs — do not guess |

- For **Astro** and **Eleventy**, do not adapt the React or Vanilla examples — consult
  the [Storybook documentation](https://storybook.js.org/docs) for the correct setup
  for those frameworks before proceeding
- After init, configure `.storybook/preview.js` following the **Storybook configuration**
  section above — this is required for design tokens and dark mode to work in stories
- Verify the `stories` glob in `.storybook/main.js` matches the project's file layout
  (see Storybook configuration above)
- Add `storybook-static/` to `.gitignore` — this is the build output directory and
  must not be committed. Do not add `.storybook/` to `.gitignore` — that is the config
  directory and must be committed
- Add the following scripts to `package.json`:
  - `"storybook": "storybook dev -p 6006"`
  - `"build-storybook": "storybook build"`
- Write at least one story per component, layout, and page that has a spec with status `Ready` or `Complete`
- Story files should be co-located with the file they describe:
  - `src/components/<Name>/<Name>.stories.{js|jsx|ts|tsx|svelte}`
  - `src/layouts/<Name>/<Name>.stories.{js|jsx|ts|tsx|svelte}`
  - `src/pages/<Name>/<Name>.stories.{js|jsx|ts|tsx|svelte}`
- Include `tags: ['autodocs']` in every story's default export to enable automatic
  documentation page generation
