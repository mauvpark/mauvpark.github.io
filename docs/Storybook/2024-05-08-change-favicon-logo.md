---
layout: post
title: How to change favicon and logo in Storybook 8
description: change favicon, change logo, Storybook 8
date: 2024-05-08 00:00:00 +0900
parent: Storybook
categories: Storybook
nav_order: 3
comments: false
---

_2024-05-08 작성_

# How to change favicon and logo in Storybook 8

{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

---

I had struggled to change favicon and logo in a storybook project.

So I want to share my tip who wants to change them.

## 1. Storybook version

My storybook version is **8.0.9**

## 2. How To

**First**, go to `.storybook/main.ts`. We will change [staticDirs](https://storybook.js.org/docs/configure/images-and-assets#serving-static-files-via-storybook-configuration) which sets a list of directories of static files to be loaded by Storybook server.

```typescript
// .storybook/main.ts
// Replace your-framework with the framework you are using (e.g., react-webpack5, vue3-vite)
import type { StorybookConfig } from "@storybook/your-framework";

const config: StorybookConfig = {
  framework: "@storybook/your-framework",
  stories: ["../src/**/*.mdx", "../src/**/*.stories.@(js|jsx|mjs|ts|tsx)"],
  staticDirs: ["../public"], //👈 Configures the static asset folder in Storybook
};

export default config;
```

**Second**, go to `public` folder and paste your icon file and logo file.

It should be as follows...

- public
  - favicon.ico
  - logo.jpg

Then, Storybook server will read `favicon.ico` and it will render your image on a tab in a browser. Then, let's try out to show some our logo.

**Third**, before we go to `.storybook/manager.ts` for applying the logo, We need a [theme](https://storybook.js.org/docs/configure/theming#create-a-theme-quickstart). Let's make my theme.

```typescript
// .storybook/YourTheme.ts
import { create } from "@storybook/theming/create";

export default create({
  base: "light",
  brandTitle: "My custom Storybook",
  brandUrl: "https://example.com",
  brandImage: "./logo.jpg", // or you could put an url from CDN.
  brandTarget: "_self",
});
```

There's our logo url `./logo.jpg`, It will read from `public` folder. After that, we need to apply our theme in `manager.ts`.

```typescript
// .storybook/manager.ts
import { addons } from "@storybook/manager-api";
import theme from "./YourTheme";

addons.setConfig({
  theme,
});
```

All done! Check out your logo and favicon are at right place. If you have any problem with this tip, please let me know by issuing in my github.

[Issue page](https://github.com/mauvpark/mauvpark.github.io/issues)
