---
name: Theme Submission
about: Submit a CSS theme to be included in the Skin Manager store
title: "[THEME] "
labels: theme-submission
assignees: ''
---

**Theme name**

**Author**

**Description**
<!-- One or two sentences shown on the theme card in the store -->

**Jellyfin compatibility**
<!-- e.g. 10.10+, 10.11+ -->

**Source repository**

**CSS URL**
<!-- Direct link to the compiled CSS file. jsDelivr CDN links are preferred for stability. -->

**Preview image URL**
<!-- A 16:9 screenshot hosted in your repo or on imgur -->

**Tags**
<!-- Pick from: dark, light, minimal, modern, colorful, backdrops, mobile-friendly, tv, icons, animations, developer -->

---

**Variables** *(optional)*
<!-- If your theme supports configurable variables, list them below.
     Leave this section blank if the theme has no variables.
     
     Each variable needs: key (UPPER_SNAKE_CASE), name, description, type, and default.
     Available types: text, color, number, boolean
     
     The key is converted to a CSS custom property automatically:
     ACCENT_COLOR -> --accent-color
     FONT_SIZE    -> --font-size
     
     Your CSS file should reference these via var(--key-name).
     Do not declare :root defaults in the CSS file itself — use the default field here instead.
-->

```json
[
  {
    "key": "EXAMPLE_COLOR",
    "name": "Example Color",
    "description": "Brief description shown in the configure popup.",
    "type": "color",
    "default": "#00a4dc"
  }
]
```

---

**Checklist**

- [ ] CSS is hosted at a stable, public URL
- [ ] Theme works on Jellyfin 10.10 or 10.11
- [ ] Theme applies with a single CSS import
- [ ] Source repository is publicly accessible
- [ ] If vars are included: CSS uses `var(--key-name)` with no `:root` defaults in the CSS file
