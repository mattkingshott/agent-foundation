# Code Style Guide - TailwindCSS Rules

All generated TailwindCSS code must match these patterns to ensure consistency and maintainability.

---

### Utility Order

Classes should follow this order:

1. **Backgrounds**: `bg-white`, `text-neutral-600`, `border-neutral-300`
2. **Borders**: `border`, `border-neutral-300`, `rounded-md`
3. **Layout**: `flex`, `grid`, `block`, `hidden`,
4. **Display**: `flex-col`, `items-center`, `justify-between`
5. **Spacing**: `space-x-2`, `gap-4`
6. **Sizing**: `w-full`, `h-38px`, `size-5`, `max-w-500px`
7. **Typography**: `font-600`, `text-15px`, `leading-normal`, `tracking-normal`
8. **Positioning**: `relative`, `absolute`, `inset-0`, `top-0`, `left-0`
9. **Effects**: `shadow`, `opacity-50`
10. **Transitions**: `transition`, `duration-300`
11. **Padding**: `p-4`, `px-5`, `py-3`,
12. **Margin**: `m-4`, `mx-5`, `my-3`,

States and responsive variants should appear after their associated utility e.g.

```html
<div class="bg-white sm:bg-neutral-100 hover:bg-neutral-300 sm:hover:bg-red-300"></div>
```

### Spacing Scale

Use precise pixel values from theme:
- Small: `px-3`, `py-2`, `gap-2`, `space-x-2`
- Medium: `px-5`, `py-4`, `gap-4`, `space-y-6`
- Large: `px-8`, `py-6`, `gap-8`, `mt-12`
- Custom: `px-14px`, `py-18px`, `gap-22px`

### Color Palette

Frequently-used colors which are good defaults.

**Primary Colors**:
- Neutral: `neutral-50`, `neutral-300`, `neutral-400`, `neutral-500`, `neutral-600`, `neutral-800`, `neutral-900`
- Sky/Blue: `sky-50`, `sky-600`, `sky-700`, `sky-900`
- Red: `red-600`, `red-700`
- Emerald: `emerald-600`
- Yellow: `yellow-500`, `yellow-600`

**Opacity Modifiers**:
- Borders: `border-neutral-300/70`, `border-sky-900/20`
- Backgrounds: `bg-black/50`, `bg-sky-500/6`
- Text: `text-neutral-500/80`

### Responsive Breakpoints

```html
<div class="hidden sm:flex md:grid lg:max-w-1050px xl:px-24"></div>
```

**Breakpoints**:
- `sm:` - Small screens (640px+)
- `md:` - Medium screens (768px+)
- `lg:` - Large screens (1024px+)
- `xl:` - Extra large screens (1280px+)