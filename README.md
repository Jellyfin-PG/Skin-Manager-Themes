# Skin Manager: Theme Repository

This repository hosts the community theme list for the [Skin Manager](https://github.com/Jellyfin-PG/Skin-Manager) Jellyfin plugin. The `skins.json` file is fetched live by the plugin, so approved themes appear in the store without requiring a plugin update.

---

## Submitting a Theme

Themes are added through GitHub issues. Open a new issue and select the **Theme Submission** template. Fill in all required fields and a maintainer will review it.

**Before submitting, make sure your theme meets these requirements:**

- The CSS is hosted at a stable, publicly accessible URL (jsDelivr CDN is strongly preferred)
- It has been tested on Jellyfin 10.10 or 10.11
- The source is publicly available (GitHub repository or equivalent)

Themes that are abandoned, broken, or inaccessible will not be accepted.

> **Note on hosting:** jsDelivr (`cdn.jsdelivr.net`) is the recommended host. It serves CSS with correct MIME types and is globally cached. Raw GitHub URLs (`raw.githubusercontent.com`) and gist URLs also work but are slower and may have availability issues.

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

### Version field

The `version` field is required. Skin Manager uses it to detect updates and manage the browser cache. When you ship a new version of your CSS, bump the version in `skins.json` and the plugin will automatically fetch the updated stylesheet and refresh it for all users without them needing to do anything.

Use any consistent versioning scheme, semantic (`1.2.0`), date-based (`25.12.31`), or a channel label (`latest`). If you use `latest` the auto-update check is skipped since there is no version to compare against.

---

## Theme Variables

Themes can expose configurable values that users can change from the Skin Manager settings page without editing any CSS. When a theme has variables, a configure button appears on the selected theme card in the store.

Variables are declared in `skins.json` using a `vars` array. The theme CSS references them using standard CSS custom properties (`var(--keyName)`). Skin Manager injects the user's saved values as a `:root` block directly into the page HTML before the stylesheet loads, so they apply instantly with no flash of unstyled content.

### Declaring variables in `skins.json`

Add a `vars` array to the theme entry. Each var needs a `key`, `name`, `description`, `type`, and `default`:

```json
{
  "name": "My Theme",
  "version": "1.0.0",
  "cssUrl": "https://cdn.jsdelivr.net/gh/author/theme@main/theme.css",
  "vars": [
    {
      "key": "accentColor",
      "name": "Accent Color",
      "description": "Primary highlight color used throughout the theme.",
      "type": "color",
      "default": "#00a4dc"
    },
    {
      "key": "fontSize",
      "name": "Base Font Size",
      "description": "Base font size. Use CSS units e.g. 14px, 1em.",
      "type": "text",
      "default": "14px"
    },
    {
      "key": "enableBackdrops",
      "name": "Show Backdrops",
      "description": "Toggle backdrop images on the home screen.",
      "type": "boolean",
      "default": "false"
    }
  ]
}
```

**Available var types:** `text` `color` `number` `boolean`

The `key` uses `camelCase`. Skin Manager converts it to `--camelCase` automatically — `accentColor` becomes `--accentColor`, `fontSize` becomes `--fontSize`, and so on.

### Using variables in your CSS

Reference the custom properties using `var()`. Always include a fallback value — this is what users on older plugin versions will see, and it ensures your theme looks correct even if no value has been saved yet:

```css
.headerTop {
  background-color: var(--accentColor, #00a4dc);
}

body {
  font-size: var(--fontSize, 14px);
}
```

### How injection works

When a user saves their variable values, Skin Manager injects a `<style>` tag directly into `index.html` before the page is served:

```html
<style id="skinmanager-vars">
:root {
    --accentColor: #e5534b;
    --fontSize: 16px;
}
</style>
```

This tag is applied immediately as the browser parses the HTML, before any JavaScript runs or any network request is made for the stylesheet.

---

## Addon CSS Sheets

Themes can include optional addon stylesheets that users enable or disable using boolean variables. This lets you ship modular features — media player reskins, third-party plugin support, alternate layouts, without loading CSS that the user does not want.

### How it works

Add a special comment to your theme CSS file for each addon:

```css
/* @sm-import-if VAR_KEY https://cdn.jsdelivr.net/gh/author/theme@main/addons/media-bar.css */
```

When Skin Manager loads your theme, it reads these comments. If `varKey` is `true` in the user's saved vars, the addon URL is fetched and appended to the theme CSS. If it is `false` or unset, the comment is stripped and nothing is loaded.

Declare the corresponding var as a `boolean` type in `skins.json`:

```json
{
  "key": "mediaSupportBar",
  "name": "Media Bar Plugin Support",
  "description": "Enable additional styles for the Media Bar plugin.",
  "type": "boolean",
  "default": "false"
}
```

### Full example

`skins.json` entry:

```json
{
  "name": "My Theme",
  "version": "1.2.0",
  "cssUrl": "https://cdn.jsdelivr.net/gh/author/theme@main/theme.css",
  "vars": [
    {
      "key": "accentColor",
      "name": "Accent Color",
      "description": "Primary highlight color.",
      "type": "color",
      "default": "#00a4dc"
    },
    {
      "key": "mediaBarSupport",
      "name": "Media Bar Plugin Support",
      "description": "Enable additional styles for the Media Bar plugin.",
      "type": "boolean",
      "default": "false"
    },
    {
      "key": "customCovers",
      "name": "Custom Media Covers",
      "description": "Enable alternate card cover styles.",
      "type": "boolean",
      "default": "false"
    }
  ]
}
```

`theme.css`:

```css
/* @sm-import-if mediaBarSupport https://cdn.jsdelivr.net/gh/author/theme@main/addons/media-bar.css */
/* @sm-import-if customCovers https://cdn.jsdelivr.net/gh/author/theme@main/addons/custom-covers.css */

.headerTop {
  background-color: var(--accentColor, #00a4dc);
}
```

Addon sheets can be hosted anywhere, jsDelivr is recommended for the same reasons as the main CSS.

### Backward compatibility

`@sm-import-if` comments are valid CSS comments and are silently ignored by any browser or plugin that does not understand them. Older versions of Skin Manager will simply load the theme without the addons.

---

## Backward Compatibility

All features are additive and optional:

- Themes without `vars` work exactly as before, no configure button appears
- Themes without `@sm-import-if` comments are unaffected by the addon system
- The `version` field is new but harmless to omit, auto-update is simply skipped for themes without it
- CSS `var(--key, fallback)` syntax means themes with variables still display correctly on older plugin versions using the fallback value

---

## License

The contents of this repository are licensed under [MIT](LICENSE). Individual themes remain the property of their respective authors under their own licenses.

