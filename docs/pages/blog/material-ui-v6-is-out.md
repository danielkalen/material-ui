---
title: Material UI v6 is out now 🎉
description: Material UI v6 ships with support for CSS variables, container queries, and advanced color schemes, as well as improved runtime performance and a reduced bundle size.
date: 2024-08-26T00:00:00.000Z
authors:
  [
    'aarongarciah',
    'brijeshb42',
    'diegoandai',
    'mnajdova',
    'romgrk',
    'samuelsycamore',
    'siriwatknp',
    'zanivan',
  ]
tags: ['Material UI', 'Product']
manualCard: true
---

Material UI v6 is now stable—just in time to celebrate [10 years since our initial commit](https://github.com/mui/material-ui/commit/28b768913b75752ecf9b6bb32766e27c241dbc46)!
It comes with new features, improvements, and an experimental integration for static CSS extraction.

## Table of contents

- [New features](#new-features)
  - [CSS theme variables](#css-theme-variables)
  - [Color schemes](#color-schemes)
  - [CSS media `prefers-color-scheme`](#css-media-prefers-color-scheme)
  - [Container queries](#container-queries)
  - [New method of applying styles](#new-method-of-applying-styles)
- [Improvements](#improvements)
  - [Optimized runtime performance](#optimized-runtime-performance)
  - [Revamped free templates](#revamped-free-templates)
  - [Stabilized Grid v2](#stabilized-grid-v2)
  - [Reduced package size](#reduced-package-size)
- [Experimental CSS extraction via Pigment CSS](#experimental-css-extraction-via-pigment-css)
  - [React Server Components](#react-server-components)
  - [Built-in `sx` prop support](#built-in-sx-prop-support)
- [React 19 support](#react-19-support)
- [Full-stack apps via Toolpad Core](#full-stack-apps-via-toolpad-core)
- [What's next](#whats-next)

## New features

### CSS theme variables

CSS variables have become an industry standard for the web.
Material UI v6 introduces a new `cssVariables` flag that generates CSS variables from serializable theme values such as palette, spacing, and typography.

```js
const theme = createTheme({ cssVariables: true, ... });
```

{{"component": "components/blog/material-ui-v6-is-out/ThemeTokens.js"}}

After enabling the flag, you can access the variables from the `theme.vars` object with the same structure as the theme.

<codeblock>

```js JSX
const CustomComponent = styled('div')(({ theme }) => ({
  backgroundColor: theme.vars.palette.background.default,
  color: theme.vars.palette.text.primary,
}));
```

```css CSS
.CustomComponent-ae73f {
  background-color: var(--mui-palette-background-default);
  color: var(--mui-palette-text-primary);
}
```

</codeblock>

#### Benefits

- Improved developer experience—you can see where the values are coming from in the browser's dev tools.
- Makes it possible to resolve the notorious [dark mode SSR flickering issue](https://github.com/mui/material-ui/issues/27651).
- CSS variables open many doors for integration—for example, you can work with a Material UI theme in a plain CSS file.

  ```css title="styles.css"
  .custom-card {
    background-color: var(--mui-palette-background-default);
    color: var(--mui-palette-text-primary);
    padding: var(--mui-spacing-2);
    font: var(--mui-font-body1);
  }
  ```

### Color schemes

Prior to v6, if you wanted to support light and dark modes, you would have to create two separate themes and handle the logic to switch between modes.
This can be complex and error-prone.

Material UI v6 introduces the new `colorSchemes` node to simplify this process: With just one line of code, your application supports both light and dark modes instantly.

```js
const theme = createTheme({ colorSchemes: { dark: true } });
// light is generated by default.

function App() {
  return <ThemeProvider theme={theme}>...</ThemeProvider>;
}
```

The `colorSchemes` node makes it possible for Material UI to support the following features out of the box:

- System preference detection
- Instant transitions between modes with the `disableTransitionOnChange` prop
- Persistent user preferences via local storage
- Automatic synchronization of the selected mode across browser tabs

#### A hook for toggling between modes

The new `useColorScheme` hook lets you read and update the mode in your application.

```jsx
import { useColorScheme } from '@mui/material/styles';

function ModeSwitcher() {
  const { mode, setMode } = useColorScheme();
  if (!mode) return null;
  return (
    <select onChange={(event) => setMode(event.target.mode)}>
      <option value="system">System</option>
      <option value="light">Light</option>
      <option value="dark">Dark</option>
    </select>
  );
}
```

### CSS media `prefers-color-scheme`

When [CSS variables](#css-theme-variables) and [color schemes](#color-schemes) are enabled, Material UI uses the `prefers-color-scheme` media query as the default method for generating CSS variables.

<codeblock>

```js JSX
import { createTheme, ThemeProvider } from '@mui/material/styles';

const theme = createTheme({
  cssVariables: true,
  colorSchemes: { dark: true },
});

function App() {
  return <ThemeProvider theme={theme}>...</ThemeProvider>;
}
```

```css CSS
:root {
  --mui-palette-primary-main: #1976d2;
  --mui-palette-background-default: #fff;
  ...
}

@media (prefers-color-scheme: dark) {
  :root {
    --mui-palette-primary-main: #90caf9;
    --mui-palette-background-default: #121212;
    ...
  }
}
```

</codeblock>

### Container queries

Material UI v6 introduces a [container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries) utility based on the existing `theme.breakpoints` API.

This feature lets you define styles based on the parent container's width instead of the viewport.

<codeblock>

```jsx JSX
const Component = styled('div')(({ theme }) => ({
  display: 'flex',
  flexDirection: 'column',
  gap: theme.spacing(2),
  [theme.containerQueries.up('sm')]: {
    flexDirection: 'row',
  },
  [theme.containerQueries('sidebar').up('400px')]: {
    // @container sidebar (min-width: 400px)
    flexDirection: 'row',
  },
}));
```

```css CSS
/* Simplified CSS Output */

.Component-ad83f {
  display: flex;
  flex-direction: column;
  gap: 16px;
  @container (min-width: 600px) {
    flex-direction: row;
  }
  @container sidebar (min-width: 400px) {
    flex-direction: row;
  }
}
```

</codeblock>

It also works with the `sx` prop:

<codeblock>

```jsx JSX
<Card
  sx={{
    '@sm': {
      flexDirection: 'row',
    },
    '@400/sidebar': {
      flexDirection: 'row',
    },
  }}
/>
```

```css CSS
/* Simplified CSS Output */

.MuiCard-root-dn383 {
  @container (min-width: 600px) {
    flex-direction: row;
  }
  @container sidebar (min-width: 400px) {
    flex-direction: row;
  }
}
```

</codeblock>

### New method of applying styles

The new API `theme.applyStyles` has been added for creating specific mode styles.
It's designed to replace the `theme.palette.mode === 'dark'` condition to fix the SSR flickering issue when combined with the CSS theme variables feature.

<codeblock>

```jsx After
const StyledInput = styled(InputBase)(({ theme }) => ({
  padding: 10,
  width: '100%',
  borderBottom: '1px solid #eaecef',
  ...theme.applyStyles('dark', {
    borderBottom: '1px solid #30363d',
  }),
  '& input': {
    borderRadius: 4,
    backgroundColor: '#fff',
    ...theme.applyStyles('dark', {
      backgroundColor: '#0d1117',
    }),
  },
}));
```

```jsx Before
const StyledInput = styled(InputBase)(({ theme }) => ({
  padding: 10,
  width: '100%',
  borderBottom:
    theme.palette.mode === 'dark' ? '1px solid #30363d' : '1px solid #eaecef',
  '& input': {
    borderRadius: 4,
    backgroundColor: theme.palette.mode === 'dark' ? '#0d1117' : '#fff',
  },
}));
```

</codeblock>

:::info
We provide a codemod to help you migrate from `theme.palette.mode === 'dark'` to `theme.applyStyles()`.
See the [v6 migration guide](/material-ui/migration/upgrade-to-v6/) for complete details.
:::

## Improvements

### Optimized runtime performance

[Several optimizations](https://github.com/mui/material-ui/pulls?q=is%3Apr+author%3Aromgrk+is%3Aclosed+perf+sort%3Aupdated-desc) have been made to improve the runtime performance of Material UI v6.

This is just the beginning of our performance improvements.
There are more optimizations in our pipeline, so stay tuned for further v6.x updates and the benchmark results.

### Revamped free templates

Explore the new and improved [free Material UI templates](https://mui.com/material-ui/getting-started/templates/) to see all of the new v6 features in action.
We've fully revamped the templates to demonstrate some common real-world use cases for our components, and to provide the perfect starting point for your next project.

{{"component": "components/blog/material-ui-v6-is-out/FreeTemplatesBento.js"}}

#### Custom styles

The new templates can be toggled between the default Material Design and a fully custom theme that's shared across the entire suite, so you can dig deep into the best approaches for component customization—or just grab the pieces you like best and start building from there.

{{"component": "components/blog/material-ui-v6-is-out/CustomThemeComparison.js"}}

### Stabilized Grid v2

The Grid v2 component has been stabilized and now uses the CSS gap property for creating space between grid items.
This is a huge improvement over the previous version, which used margins and padding to create space.

The responsive configuration has been consolidated into the `size` and `offset` props for consistency.

```jsx
import Grid from '@mui/material/Grid2';

<Grid container>
  <Grid size={{ xs: 6, sm: 4, lg: 3 }} />
  <Grid size={{ xs: 6, sm: 4, lg: 3 }} />
  <Grid size={{ xs: 6, sm: 4, lg: 3 }} />
</Grid>;
```

### Reduced package size

To align with React 19's removal of UMD builds, Material UI has also removed its UMD bundle.
This results in a reduction of the `@mui/material` package size by 2.5MB, or 25% of the total size in v5.
See [Package Phobia](https://packagephobia.com/result?p=@mui/material) for more details.
Instead of UMD, we recommend using ESM-based CDNs such as [esm.sh](https://esm.sh/).
For alternative installation methods, refer to the [CDN documentation](/material-ui/getting-started/installation/#cdn).

## Experimental CSS extraction via Pigment CSS

Material UI v5 uses Emotion as its default styling solution.
Because it's a runtime CSS-in-JS library, Emotion forces us to make compromises to performance and bundle size.

Material UI v6 introduces opt-in integration with [Pigment CSS](https://github.com/mui/pigment-css), our new zero-runtime styling library that eliminates the runtime overhead while preserving the API design patterns you already know and love.

:::warning
Pigment CSS integration is still in the experimental stage.
We recommend testing it out with our examples ([Next.js](https://github.com/mui/material-ui/tree/master/examples/material-ui-pigment-css-nextjs-ts) or [Vite](https://github.com/mui/material-ui/tree/master/examples/material-ui-pigment-css-vite)), and we'd love to get your feedback to help us improve it.

When you're ready to integrate Pigment CSS into your project, you can find complete details in [the guide to migrating to Pigment CSS](/material-ui/migration/migrating-to-pigment-css/).
:::

### React Server Components

Once integrated with Pigment CSS, Material UI v6 provides a separate set of layout components—Grid, Container, and Stack—that are compatible with React Server Components.

```jsx
import Grid from '@mui/material-pigment-css/Grid';
import Container from '@mui/material-pigment-css/Container';
import Stack from '@mui/material-pigment-css/Stack';
```

### Built-in sx prop support

With Pigment CSS integration, all JSX elements support the `sx` prop out of the box.
This means you no longer have to clutter the DOM with wrappers like Box or Stack in order to apply custom styles to a `<div>` or any other element with the `sx` prop.

```diff
-import Box from '@mui/material/Box';

-<Box component="img" sx={{ padding: 2 }} />
+<img sx={{ padding: 2 }} />
```

## React 19 support

Material UI v6 is ready for React 19.
We've been testing Material UI with the latest [React 19 RC](https://react.dev/blog/2024/04/25/react-19) versions to ensure compatibility once the stable release of React 19 is out.

We're also working on backporting React 19 support to Material UI v5—stay tuned.

## Full-stack apps via Toolpad Core

To help you build full-stack apps faster than ever, we're launching [Toolpad Core](https://mui.com/toolpad/), a framework for building dashboards and internal tools with Material UI and Next.js that comes with routing, layout components, and authentication already configured for you.

Stay tuned for more updates on Toolpad Core in the near future.

## Get started with Material UI v6

Ready to upgrade to Material UI v6?
Head to [the v6 migration guide](/material-ui/migration/upgrade-to-v6/) next.

Visit the links below for further details on some of the key features covered here:

- [Pigment CSS integration](/material-ui/migration/migrating-to-pigment-css/)
- [Container queries](/material-ui/customization/container-queries/)
- [CSS theme variables](/material-ui/customization/css-theme-variables/overview/)
- [Stabilized Grid v2](/material-ui/migration/upgrade-to-v6/#grid2)
- [Free templates](/material-ui/getting-started/templates/)