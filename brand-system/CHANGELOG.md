# WDS Changelog

كلّ تَغيير في النِظام يُسَجَّل هنا. نَتبَع [Semantic Versioning](https://semver.org).

---

## [1.0.0] — 2026-05-12

### المُضاف
- النِظام الأساسيّ كاملاً: 8 طَبقات، 40+ وثيقة.
- 01-strategy: purpose, positioning, personas, narrative, values.
- 02-foundations: colors, typography, spacing, grid, iconography, imagery, motion, elevation.
- 03-components: buttons, inputs, cards, badges, modals, tables, forms, navigation, breadcrumbs, pagination, tabs, accordions, tooltips, alerts, toasts, progress.
- 04-patterns: page-layouts, hero, empty/error/loading/success states, onboarding, search, filter, dataviz.
- 05-voice-and-tone: voice principles, tone by context, microcopy library, error messages, empty copy, CTA library, notification copy, SEO.
- 06-accessibility: WCAG, keyboard, screen readers, color contrast.
- 07-internationalization: RTL, Arabic typography, translation strategy.
- 08-governance: contribution, review, versioning, deprecation.

### الكود المُطبَّق
- `frontend/tailwind.config.ts` — كل الـ tokens (colors, type, spacing, shadows, motion).
- `frontend/app/[locale]/design-system/page.tsx` — صَفحة عَرض حيّة لكلّ المُكوّنات.
- `frontend/styles/tokens.css` — CSS variables (light + dark — مَحجوزة لـ v1.1).

---

## القادم (v1.1)
- Dark mode tokens.
- مُكوّن DataTable مَعَ filtering server-side.
- مُكوّن Calendar (للجَدوَلَة).
- صَفحات `examples/*` لكلّ pattern في الواقع.
