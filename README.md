# Skin Manager — Theme Repository

This repository hosts the community theme list for the [Skin Manager](https://github.com/Jellyfin-PG/Skin-Manager) Jellyfin plugin. The `skins.json` file is fetched live by the plugin, so approved themes appear in the store without requiring a plugin update.

---

## Submitting a Theme

Themes are added through GitHub issues. Open a new issue and select the **Theme Submission** template. Fill in all required fields and a maintainer will review it.

**Before submitting, make sure your theme meets these requirements:**

- The CSS is hosted at a stable, publicly accessible URL (jsDelivr CDN or GitHub raw are preferred)
- The theme works with a single `@import` or stylesheet link — no additional files required
- It has been tested on Jellyfin 10.10 or 10.11
- The source is publicly available (GitHub repository or equivalent)

Themes that are abandoned, broken, or require files beyond a single CSS import will not be accepted.

---

## Theme Format

Each entry in `skins.json` uses the following structure:

```json
{
  "name": "ThemeName",
  "author": "AuthorHandle",
  "description": "One or two sentences describing the theme.",
  "version": "1.0.0",
  "jellyfin": "10.11+",
  "tags": ["dark", "minimal"],
  "previewUrl": "https://example.com/preview.png",
  "sourceUrl": "https://github.com/author/theme",
  "cssUrl": "https://cdn.jsdelivr.net/gh/author/theme@main/theme.css"
}
```

**Available tags:** `dark` `light` `minimal` `modern` `colorful` `backdrops` `mobile-friendly` `tv` `icons` `animations` `developer`

---

## Theme Variables

Themes can expose configurable values that users can change from the Skin Manager settings page without editing any CSS. When a theme has variables, a configure button appears next to the Select button in the store.

Variables are declared in `skins.json` using a `vars` array on the theme entry. The theme CSS file references them using standard CSS custom properties (`var(--key-name)`). Skin Manager injects a `:root` block with the user's saved values before the theme stylesheet is loaded.

### How to add variables to a theme

**1. Declare vars in `skins.json`**

Add a `vars` array to the theme entry. Each var needs a `key`, `name`, `description`, `type`, and `default`:

```json
{
  "name": "My Theme",
  "cssUrl": "https://cdn.jsdelivr.net/gh/author/theme@main/theme.css",
  "vars": [
    {
      "key": "ACCENT_COLOR",
      "name": "Accent Color",
      "description": "Primary highlight color used throughout the theme.",
      "type": "color",
      "default": "#00a4dc"
    },
    {
      "key": "FONT_SIZE",
      "name": "Base Font Size",
      "description": "Base font size in pixels.",
      "type": "number",
      "default": "14"
    },
    {
      "key": "SHOW_BACKDROPS",
      "name": "Show Backdrops",
      "description": "Toggle backdrop images on the home screen.",
      "type": "boolean",
      "default": "true"
    }
  ]
}
```

**Available var types:** `text` `color` `number` `boolean`

The `key` field uses `UPPER_SNAKE_CASE`. Skin Manager converts it to `--lower-kebab-case` automatically when injecting — so `ACCENT_COLOR` becomes `--accent-color`, `FONT_SIZE` becomes `--font-size`, and so on.

**2. Use `var()` in your CSS file**

Reference the custom properties in your CSS. Do not declare `:root` defaults in the CSS file — all defaults live in `mods.json` via the `default` field. Skin Manager's injected `:root` block is the only source of truth.

```css
/* theme.css */

:root {
  /* Do not put defaults here — declare them in skins.json instead */
}

.headerTop {
  background-color: var(--accent-color);
}

.cardText {
  font-size: var(--font-size, 14px); /* fallback is optional but harmless */
}
```

**3. How it gets injected**

When a user saves their variable values, Skin Manager injects this into `index.html` before the theme stylesheet:

```html
<style id="skinmanager-vars">
:root {
    --accent-color: #e5534b;
    --font-size: 16;
    --show-backdrops: false;
}
</style>
<style id="skinmanager-theme">
@import url("https://cdn.jsdelivr.net/.../theme.css");
</style>
```

The `:root` block is in a separate `<style>` tag before the `@import` — this is required because `@import` must be the first rule in any stylesheet.

### Backward compatibility

Variables are fully optional. Themes without a `vars` array work exactly as before — no configure button appears, nothing changes for existing users. Adding vars to an existing theme is non-breaking.

---

## License

The contents of this repository are licensed under [MIT](LICENSE). Individual themes remain the property of their respective authors under their own licenses.
