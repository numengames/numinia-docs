# Numengames Website Developer Guide

Welcome to the Numengames website project! Whether you're a seasoned team member or just joining us on this journey, this guide will be your compass through our codebase. We've designed this documentation to help you understand not just what our code does, but why we've structured it the way we have.

## Project Overview

The Numengames website represents our digital front door - a place where users first experience our brand and products. To create a seamless, performant experience, we've built it using:
- **Astro** for static site generation and page routing
- **Svelte** for interactive components
- **TypeScript** for type safety
- **Tailwind CSS** for styling

We've embraced a component-based architecture with clear separation of concerns, allowing us to build features collaboratively while maintaining consistency.

## Directory Structure

Our project follows a thoughtful organization that reflects how components and features relate to each other:

```
src/
├── components/
│   ├── ui/             # Reusable UI components
│   │   ├── buttons/    # Button components
│   │   ├── typography/ # Text components (Heading, Paragraph)
│   ├── layout/         # Layout components
│   │   ├── sections/   # Section wrappers
│   ├── features/       # Feature-specific components
│   │   ├── hero/       # Hero section components
│   │   │   ├── HeroSection.astro
│   │   │   ├── HeroContent.astro
├── hooks/              # Custom Svelte hooks
├── stores/             # Svelte stores for state management
├── layouts/            # Page layout templates
├── pages/              # Page components and routes
├── i18n/               # Internationalization
├── styles/             # Global styles
└── types/              # TypeScript type definitions
```

Think of our codebase as a city - UI components are the building blocks, layouts create the streets and infrastructure, and features are the distinctive neighborhoods that make our city unique.

## Component Organization

### UI Components

Like the atoms and molecules in a design system, our UI components in `src/components/ui/` are the fundamental building blocks. They're organized by category to make them easy to discover:

- **Typography**: Text components such as `Heading.astro` and `Paragraph.astro` ensure consistent text presentation
- **Buttons**: Button components like `Button.astro` provide interactive elements with consistent styling

When working with UI components, remember they should be:
- **Pure**: Like good ingredients in a recipe, they should do one thing well with minimal complexity
- **Reusable**: Designed to work across multiple features without modification
- **Configurable**: Flexible enough to adapt to different needs through props

### Layout Components

In `src/components/layout/`, you'll find the structural elements that give our pages their shape:

- **Section.astro**: A wrapper that provides consistent spacing, width constraints, and structural elements

Think of layout components as the blueprints for our site - they define spaces but don't determine what goes inside them. They should:
- Provide consistent structure across the application
- Handle responsive layouts gracefully
- Remain agnostic to the specific content they contain

### Feature Components

This is where our website comes to life. In `src/components/features/`, we combine UI and layout components to create meaningful user experiences, organized by functionality:

- **HeroSection.astro**: The welcoming face of a page that captures attention
- **HeroContent.astro**: The specific messaging and call-to-action within the hero

When building feature components, imagine them as complete experiences that should:
- Tell a coherent story about a specific functionality
- Combine UI and layout components into something greater than the sum of its parts
- Handle any feature-specific logic or state

## Styling with Tailwind CSS

### Configuration

Colors, spacing, typography - these design decisions define our visual identity. We've captured them in `tailwind.config.cjs` at the root of the project:

```js
// Example of color configuration in tailwind.config.cjs
colors: {
  primary: {
    beige: '#F9EBDC',
    coralRed: '#F35059',
    darkRed: '#D33440',
    panther: '#212123',
    mediterranean: '#A6DAD5'
  },
  light: {
    background: '#F9F9F9',
    text: '#212123',
    accent: '#F35059',
    muted: '#717171',
  },
  dark: {
    background: '#212123',
    text: '#F9EBDC',
    accent: '#F35059',
    muted: '#909091',
  }
}
```

> **Important**: Our default persona is nocturnal! The project currently uses dark mode as the default theme. The `dark` class is applied to the HTML element by default, with our theme switcher allowing users to step into the light if they prefer.

### Global Styles

Our global styling foundation in `src/styles/global.css` includes:

- Base styles and Tailwind directives that set the stage
- Font declarations that give our text its voice
- Theme-specific styles that adapt to light and dark modes
- Utility classes that solve common styling challenges

### Component Styling Best Practices

When styling components, follow these principles:

1. **Embrace Tailwind's utility-first approach** - it keeps styles close to the components they affect
2. **Create consistent component variants** using the `className` prop pattern to maintain predictability
3. **Lean on theme variables** instead of hard-coding values, making site-wide changes more manageable
4. **Design with both light and dark in mind** using our theme variables

### CSS Custom Components

For styles that deserve to be reused, we've created custom component classes in the `@layer components` section of `global.css`:

```css
@layer components {
  .hero-title {
    @apply text-6xl leading-tight 2xl:text-7xl 2xl:leading-tight font-medium tracking-[-0.03em];
  }
  
  .theme-card {
    @apply bg-light-background border border-light-muted/20 dark:bg-dark-background dark:border-dark-muted/20;
  }
}
```

These act as shortcuts for frequently used style combinations, helping us maintain consistency while reducing repetition.

## State Management

### Stores

Not everything belongs in component state. For information that needs to be accessible across the application, we use Svelte stores in `src/stores/`:

- **theme.ts**: Remembers whether the user prefers to walk in darkness or light
- **locale.ts**: Keeps track of which language is currently speaking to the user

All stores are conveniently re-exported from `index.ts`, saving you from import gymnastics.

## Hooks

Custom Svelte hooks in `src/hooks/` package reusable logic that components can tap into:

- **useMediaQuery.ts**: Listens for changes in viewport size so components can adapt

Each hook should focus on telling one story well - handling a single responsibility and sharing its wisdom with any component that asks.

## Internationalization (i18n)

Our website speaks multiple languages through:
- URL-based locale routing (`/[locale]/page`) that identifies the user's language preference
- Translation objects in the `i18n/` directory that provide the vocabulary
- A locale store that remembers the current language setting

## Best Practices

### Component Organization

1. **Follow the Component Hierarchy**:
   Start with the basics (UI), add structure (layout), then create experiences (features) - this mental model helps maintain clarity.

2. **Component Naming**:
   - PascalCase for component names makes them stand out in imports
   - Suffixes like `.astro` and `.svelte` clearly identify the component type
   - Descriptive names tell developers what the component does before they even open the file

3. **File Organization**:
   - Group related components in feature-specific directories to tell a cohesive story
   - Use index files to simplify imports and keep your code cleaner

### State Management

1. **Local vs. Global State**:
   - Component state is like personal belongings - keep them close when they're only needed locally
   - Stores are for information that needs to travel - use them sparingly but confidently for truly global state

2. **Store Organization**:
   - Each store should tell one clear story about one aspect of application state
   - The index.ts re-export pattern keeps imports tidy across the application

### TypeScript

1. **Type Definitions**:
   - Shared types in the `types/` directory create a common language for the entire codebase
   - Specific types for component props set clear expectations about what data components need
   - Avoiding `any` prevents the frustrating bugs that come from type ambiguity

### Responsive Design

1. **Mobile-First Approach**:
   - We design for the smallest screen first, then enhance for larger viewports
   - The `useMediaQuery` hook helps components adapt their behavior to different screen sizes
   - Tailwind's responsive classes make responsive styling intuitive and visual

## Adding New Features

When you're tasked with bringing a new feature to life, follow this narrative arc:

1. **Identify UI Components**:
   - Start by looking through our existing UI components - can they tell your story?
   - Only add new UI components when you need a building block that doesn't exist yet

2. **Create Feature Components**:
   - Carve out a home for your feature in `components/features/`
   - Combine existing UI and layout components to craft your user experience

3. **Add State Management**:
   - Determine if your feature needs to remember things beyond its own boundaries
   - If so, add a store that manages just what you need, nothing more

4. **Add Internationalization**:
   - Ensure your feature can speak to users in all the languages we support
   - Add translations that maintain the tone and meaning across languages

## Resources and References

### Design Resources

- **Figma Design**: Explore our visual language and component specifications in [Figma](https://www.figma.com/design/a3KJNnhpZBvoDLaTErD1lr/Numen-Web?node-id=2006-1743&p=f&t=tLLxJxD3pBVc6Ch3-0)
- **Brand Guidelines**: The colors, typography, and styles that give our brand its distinctive voice

### Development References

- **Astro Documentation**: [https://docs.astro.build](https://docs.astro.build)
- **Svelte Documentation**: [https://svelte.dev/docs](https://svelte.dev/docs)
- **Tailwind CSS Documentation**: [https://tailwindcss.com/docs](https://tailwindcss.com/docs)

## Conclusion

This guide isn't just documentation - it's a shared understanding of how we build together. By following these patterns and practices, we create a codebase that's greater than the sum of its parts - one that's maintainable, extensible, and a joy to work with.

As you navigate through our components and features, remember that everything has its place and purpose. The clear separation of UI, layout, and feature components creates a natural workflow that makes it easier to understand existing code and contribute new features with confidence.

Welcome to the team - we're excited to see what you'll build with us!