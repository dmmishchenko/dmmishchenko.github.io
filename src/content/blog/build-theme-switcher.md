---
title: "Building a Modern Theme Switcher in Angular"
description: "Learn how to implement a robust theme switching system in Angular that supports light, dark, and system themes with best practices and modern CSS features"
pubDate: "Mar 29 2025"
heroImage: "/theme-switcher.jpeg"
---

> **TL;DR:** This guide shows how to build a modern theme switcher in Angular using Signals, CSS variables, and the new `light-dark()` function. We'll cover implementation details, best practices, and how to make it easily adoptable across your application.


In this guide, we'll explore how to implement a robust theme switching system in Angular that supports light, dark, and system themes. We'll cover the implementation details, best practices, and how to make it easily adoptable across your entire application.

> 💡 You can find the complete source code for this implementation on [GitHub](https://github.com/dmmishchenko/theme-switcher).

## Prerequisites

Before diving into the implementation, make sure you have:

- Angular 19+ installed
- Basic understanding of Angular components and services
- Familiarity with CSS variables and modern CSS features
- Node.js 18+ and npm installed

### Who this article is for

- Angular developers looking to implement a robust theming system
- Developers who want to support system theme preferences
- Teams needing a maintainable and performant theme solution
- Anyone interested in modern Angular and CSS features

## Table of Contents

- [Overview](#overview)
- [Implementation Approach](#implementation-approach)
- [CSS Variables and light-dark() Function](#css-variables-and-light-dark-function)
- [Theme Switcher Component](#theme-switcher-component)
- [System Theme Integration](#system-theme-integration)
- [Component Adoption](#component-adoption)
- [Forcing Theme Sections](#forcing-theme-sections)
- [Best Practices](#best-practices)
- [Common Pitfalls to Avoid](#common-pitfalls-to-avoid)
- [Testing and Browser Compatibility](#testing-and-browser-compatibility)
- [Conclusion](#conclusion)
- [Further Reading](#further-reading)

## Overview

A modern theme switcher should:

- Support light and dark themes
- Respect system preferences
- Persist user choices
- Be easy to maintain
- Work seamlessly across components
- Provide smooth transitions

## Implementation Approach

Our implementation uses three key technologies:

1. Angular Signals for state management
2. CSS Variables for theme definition
3. The `light-dark()` CSS function for theme values

### Why This Approach?

- **Signals**: Provide reactive state management without complex state libraries
- **CSS Variables**: Enable dynamic theme switching without JavaScript overhead
- **light-dark()**: Simplifies theme value definition and maintenance

## CSS Variables and light-dark() Function

The core of our theming system uses CSS variables with the `light-dark()` function:

```css
:root {
  color-scheme: light dark;
  --accent: light-dark(#2337ff, #7c89ff);
  --black-raw: light-dark(rgb(15, 18, 25), rgb(255, 255, 255));
  --gray: light-dark(rgb(96, 115, 159), rgb(171, 178, 191));
  --background: light-dark(#fff, #1a1b26);
}
```

Benefits of this approach:

- Single source of truth for color values
- Automatic system theme support
- Easy to maintain and update
- No need for separate theme files

## Theme Switcher Component

The theme switcher component manages theme state and user preferences:

```typescript
export class ThemeSwitcherComponent implements OnInit, OnDestroy {
  // Track system theme preferences
  private readonly prefersColorScheme = window.matchMedia(
    "(prefers-color-scheme: dark)"
  );

  // Reactive state with signals
  protected readonly currentTheme = signal<Theme>(
    (localStorage.getItem("theme") as Theme) || "system"
  );

  // Computed theme value
  protected readonly effectiveTheme = computed(() => {
    if (this.currentTheme() === "system") {
      return this.prefersColorScheme.matches ? "dark" : "light";
    }
    return this.currentTheme();
  });
}
```

Key features:

- Uses signals for reactive state
- Persists preferences in localStorage
- Computes effective theme based on system preference
- Clean-up with OnDestroy

<details id="stackblitz-details">
<summary>🎮 Try it out on Stackblitz</summary>
<div class="stackblitz-container">
  <iframe src="https://stackblitz.com/edit/github-cjbtqt8u?ctl=1&embed=1&file=src%2Fapp%2Ftheme-switcher%2Ftheme-switcher.component.ts" style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;" title="Theme Switcher Example"></iframe>
</div>
</details>

<script>
  // Simple A/B testing - randomly set initial state
  const details = document.getElementById('stackblitz-details');
  details.open = Math.random() > 0.5; // 50% chance of being open
</script>

---

## System Theme Integration

System theme support is implemented at two levels:

1. **CSS Level** using `prefers-color-scheme`:

```css
@media (prefers-color-scheme: dark) {
  /* Dark theme styles */
}
```

2. **JavaScript Level** using MediaQueryList:

```typescript
window
  .matchMedia("(prefers-color-scheme: dark)")
  .addEventListener("change", (e) => {
    if (currentTheme === "system") {
      updateTheme("system");
    }
  });
```

## Component Adoption

Components can easily adopt theming by using CSS variables:

```scss
.card {
  background: var(--background);
  color: var(--black-raw);
  border: 1px solid var(--gray-light);

  &:hover {
    border-color: var(--accent);
    box-shadow: var(--box-shadow);
  }
}
```

Benefits:

- No component-specific theme logic needed
- Automatic theme switching
- Consistent look across the application
- Easy to maintain and update

## Forcing Theme Sections

While our theme switcher respects system preferences and user choices globally, sometimes you might need specific sections of your application to maintain a particular theme regardless of the global setting. The `color-scheme` property enables this functionality.

### Basic Usage

```html
<section class="dark-section">
  <h2>Always Dark Section</h2>
  <p>This section will remain dark regardless of system or user preferences.</p>
</section>

<section class="light-section">
  <h2>Always Light Section</h2>
  <p>
    This section will remain light regardless of system or user preferences.
  </p>
</section>
```

```scss
.dark-section {
  color-scheme: dark;
  // The light-dark() function will use the dark value here
  background: var(--background);
  color: var(--black-raw);
}

.light-section {
  color-scheme: light;
  // The light-dark() function will use the light value here
  background: var(--background);
  color: var(--black-raw);
}
```

### Use Cases

This approach is particularly useful for:

- Marketing sections that need consistent branding
- Code snippets or documentation that should maintain readability
- Media galleries with specific visual requirements
- Third-party widget containers

### Implementation Tips

1. **Scope CSS Variables**:

```scss
.themed-section {
  color-scheme: dark;

  // Override specific variables for this section
  --background: #1a1b26;
  --text-color: #ffffff;
}
```

2. **Handle Nested Components**:

```typescript
@Component({
  selector: "app-themed-section",
  template: `
    <section [class]="forcedTheme">
      <ng-content></ng-content>
    </section>
  `,
  styles: [
    `
      .dark {
        color-scheme: dark;
      }
      .light {
        color-scheme: light;
      }
    `,
  ],
})
export class ThemedSectionComponent {
  forcedTheme: Signal<"dark" | "light"> = signal("dark");
}
```

3. **Maintain Accessibility**:

```scss
.themed-section {
  color-scheme: dark;

  // Ensure proper contrast even in forced theme
  --accent: #7c89ff; // Brighter accent for dark scheme

  // Add visual boundary for context
  border: 1px solid var(--gray-light);
  border-radius: 8px;
}
```

### Best Practices for Forced Themes

1. **Use Sparingly**:

   - Only force themes when absolutely necessary
   - Consider the user's preference first
   - Document why a section needs a forced theme

2. **Maintain Consistency**:

   - Use the same CSS variables
   - Keep transitions smooth
   - Ensure proper contrast ratios

3. **Handle Edge Cases**:
   - Test with system theme changes
   - Verify nested themed sections
   - Check interaction with global theme switches

## Best Practices

1. **Theme Definition**:

   - Use semantic variable names (e.g., `--background` instead of `--white`)
   - Group related variables
   - Document color usage

2. **Performance**:

   - Use CSS Variables for dynamic values
   - Avoid JavaScript-based theme switching
   - Implement smooth transitions

3. **Accessibility**:

   - Ensure sufficient color contrast
   - Test with screen readers
   - Support reduced motion preferences

4. **User Experience**:
   - Persist user preferences
   - Provide smooth theme transitions
   - Respect system preferences by default

## Common Pitfalls to Avoid

1. **Direct Color Usage**:

   ```scss
   // ❌ Bad
   .element {
     color: #000;
   }

   // ✅ Good
   .element {
     color: var(--black-raw);
   }
   ```

2. **Theme-Specific Styles**:

   ```scss
   // ❌ Bad
   [data-theme="dark"] .element { ... }

   // ✅ Good
   .element { color: var(--text-color); }
   ```

3. **Complex State Management**:

   ```typescript
   // ❌ Bad
   class ThemeService {
     private theme = new BehaviorSubject<Theme>('light');
   }

   // ✅ Good
   protected readonly currentTheme = signal<Theme>('system');
   ```

## Testing and Browser Compatibility

<br/>

### Testing the Theme Switcher

To ensure your theme switcher works correctly across different scenarios:

1. **Manual Testing**:
   - Test theme switching on different devices and browsers
   - Verify system theme detection
   - Check theme persistence after page reload
   - Test forced theme sections

2. **Accessibility Testing**:
   - Verify color contrast ratios meet WCAG guidelines
   - Test with screen readers
   - Check keyboard navigation
   - Validate reduced motion support

### Browser Compatibility

The `light-dark()` function is a modern CSS feature with the following browser support:

- Chrome: 123+
- Firefox: 120+
- Safari: 17.5+
- Edge: 123+

> ⚠️ **Note:** The `light-dark()` function is a relatively new feature. For older browsers, consider providing a fallback:

```css
:root {
  /* Fallback for older browsers */
  --accent: #2337ff;
  
  /* Modern browsers */
  @supports (color: light-dark(#000, #fff)) {
    --accent: light-dark(#2337ff, #7c89ff);
  }
}
```

For broader browser support, you can also use the `prefers-color-scheme` media query as a fallback:

```css
:root {
  /* Fallback using prefers-color-scheme */
  --accent: #2337ff;
  
  @media (prefers-color-scheme: dark) {
    --accent: #7c89ff;
  }
  
  /* Modern browsers with light-dark() */
  @supports (color: light-dark(#000, #fff)) {
    --accent: light-dark(#2337ff, #7c89ff);
  }
}
```

## Conclusion

Building a theme switcher with Angular's modern features and CSS variables provides a maintainable, performant, and user-friendly solution. The combination of signals for state management and CSS variables for styling makes it easy to implement and adopt across your entire application.

Remember to:

- Use CSS variables for theme values
- Leverage the `light-dark()` function
- Respect system preferences
- Maintain accessibility
- Keep the implementation simple

This approach scales well with application growth and provides a solid foundation for theme management in your Angular applications.

> 🔍 Want to explore the implementation in detail? Check out the complete source code on [GitHub](https://github.com/dmmishchenko/theme-switcher).

### Next Steps

- Implement the theme switcher in your application
- Test with different devices and browsers
- Consider adding more theme customization options
- Share your implementation with the Angular community

## Further Reading

For a deeper understanding of the concepts and technologies used in this guide, check out these excellent resources:

1. [MDN: light-dark() CSS function](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/light-dark)

   - Comprehensive documentation of the `light-dark()` function
   - Detailed syntax and usage examples
   - Browser compatibility information
   - Related CSS color concepts

2. [web.dev: Building a theme switch component](https://web.dev/articles/light-dark)
   - In-depth tutorial on implementing dark mode
   - Best practices for theme switching
   - Performance considerations
   - Accessibility guidelines

These resources provide additional context and advanced techniques for implementing robust theme switching in web applications.

---

_This article was last updated on March 29, 2025. The code examples use Angular 19._
