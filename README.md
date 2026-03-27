# Skin Manager: Theme Repository

This repository hosts the community theme list for the [Skin Manager](https://github.com/Jellyfin-PG/Skin-Manager) Jellyfin plugin. The `skins.json` file is fetched live by the plugin, so approved themes appear in the store without requiring a plugin update.

## Submitting a Theme

Themes are added through GitHub issues. Open a new issue and select the **Theme Submission** template. Fill in all required fields and a maintainer will review it.

**Before submitting, make sure your theme meets these requirements:**

* The CSS is hosted at a stable, publicly accessible URL (jsDelivr CDN is strongly preferred)

* It has been tested on Jellyfin 10.10 or 10.11

* The source is publicly available (GitHub repository or equivalent)

Themes that are abandoned, broken, or inaccessible will not be accepted.

> **Note on hosting:** jsDelivr (`cdn.jsdelivr.net`) is the recommended host. It serves CSS with correct MIME types and is globally cached. Raw GitHub URLs (`raw.githubusercontent.com`) and gist URLs also work but are slower and may have availability issues.

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
  "cssUrl": "https://cdn.jsdelivr.net/gh/author/theme@main/theme.css",
  "preconnect": [
    "https://fonts.googleapis.com",
    "https://i.imgur.com"
  ]
}
```

**Available tags:** `dark` `light` `minimal` `modern` `colorful` `backdrops` `mobile-friendly` `tv` `icons` `animations` `developer`

### Preconnect field

An optional array of origin URLs that your theme frequently contacts (like Google Fonts or image hosts). Skin Manager will dynamically inject `<link rel="preconnect">` tags for these domains to drastically accelerate the browser's DNS and TLS Handshake waterfall.

### Version field

The `version` field is required. Skin Manager uses it to detect updates and manage the browser cache. When you ship a new version of your CSS, bump the version in `skins.json` and the plugin will automatically fetch the updated stylesheet and refresh it for all users without them needing to do anything.

Use any consistent versioning scheme, semantic (`1.2.0`), date-based (`25.12.31`), or a channel label (`latest`). If you use `latest` the auto-update check is skipped since there is no version to compare against.

## Theme Variables

Themes can expose configurable values that users can change from the Skin Manager settings page without editing any CSS. When a theme has variables, a configure button appears on the selected theme card in the store.

Variables are declared in `skins.json` using a `vars` array. The theme CSS references them using standard CSS custom properties (`var(--key-name)`). Skin Manager injects the user's saved values as a highly specific `:root` block directly into the page HTML before the stylesheet loads, forcing them to override the base theme.

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
      "key": "FONT_SIZE",
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

The `key` can use `camelCase`, `PascalCase`, or `UPPER_SNAKE_CASE`. Skin Manager converts it to `--lower-kebab-case` automatically.
For example, `accentColor` becomes `--accent-color`, `FONT_SIZE` becomes `--font-size`, and so on.

### Using variables in your CSS

Reference the custom properties using `var()`. Always include a fallback value — this is what users on older plugin versions will see, and it ensures your theme looks correct even if no value has been saved yet:

```css
.headerTop {
  background-color: var(--accent-color, #00a4dc);
}

body {
  font-size: var(--font-size, 14px);
}
```

**Alternative String Substitution:**
If you want to completely bypass CSS cascading and `var()` lookups, you can use double-curly braces directly in your CSS. Skin Manager will replace these strings exactly before the stylesheet loads:

```css
.headerTop { background-color: {{accent-color}}; }
```

## Addon CSS Sheets

Themes can include optional addon stylesheets that users enable or disable using boolean variables. This lets you ship modular features — media player reskins, third-party plugin support, alternate layouts, without loading CSS that the user does not want.

### Declaring Addons

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

### Full Example

**`skins.json` entry:**

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
    }
  ],
  "preconnect": [
    "https://fonts.googleapis.com"
  ]
}
```

**`theme.css`:**

```css
/* @sm-import-if mediaBarSupport https://cdn.jsdelivr.net/gh/author/theme@main/addons/media-bar.css */

.headerTop {
  background-color: var(--accent-color, #00a4dc);
}
```

Addon sheets can be hosted anywhere, jsDelivr is recommended for the same reasons as the main CSS.

## Architecture & Injection

When a theme is enabled, Skin Manager utilizes an **Offline Disk Proxy**. 

The plugin downloads your `cssUrl` (and any enabled addons) using a robust backend connection and caches the raw stylesheets to the Jellyfin server's local storage. 

Skin Manager then processes the stylesheet either server-side (for Global Themes) or client-side (for User Themes), automatically substituting your variables natively into the CSS string. It finally injects the fully compiled, highly-optimized payload into the Jellyfin web client directly from a local API endpoint on the server (`/api/SkinManager/Resource`).

This local proxy injection guarantees:
1. Instant load times and complete immunity to external CDN outages.
2. Freedom from browser cross-origin policy blockages.
3. Guaranteed specificity precedence for your custom variables.

## Backward Compatibility

All features are additive, optional, and gracefully degrade on older plugin versions:

* **Variables**: Themes without `vars` work exactly as before; no configure button appears.
* **Fallbacks**: CSS `var(--key, fallback)` syntax ensures themes correctly render default aesthetic on older plugins.
* **Addons**: `@sm-import-if` comments are valid CSS comments and are silently ignored by older plugin versions.
* **Networking**: The `preconnect` array is gracefully ignored by older plugin versions.
* **Versioning**: The `version` field is harmless to omit; auto-update checks are simply skipped.

## License

The contents of this repository are licensed under [MIT](LICENSE). Individual themes remain the property of their respective authors under their own licenses.
