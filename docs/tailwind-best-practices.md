# Tailwind CSS Best Practices

A comprehensive guide for using Tailwind effectively and consistently across our engineering organization.

## 🎯 Core Principles

### 1. Component-First Approach

- **Use Design System Components**: Always prefer our design system components over raw JSX elements with Tailwind classes
- **Props Over Classes**: Prefer component props to control variants and styles, but use `className`/`twClassname`/`tw` when no equivalent prop exists
- **Example**:

**React Web:**
```tsx
// ❌ Don't
<div className="bg-default p-4 rounded-lg">
  Content
</div>

// ✅ Do
<Box backgroundColor="default" padding={4} borderRadius="lg">
  Content
</Box>
```

**React Native:**
```tsx
// ❌ Don't
<View style={tw`bg-default p-4 rounded-lg`}>
  Content
</View>

// ✅ Do
<Box backgroundColor="default" padding={4} borderRadius="lg">
  Content
</Box>
```

### 2. Design Token Integration

- **Use Design Token Generated Classes Only**: Always use design token based classes that are generated from our design system
- **No Default Tailwind Values**: Never use Tailwind's default color palette or typography scale (these should be removed in the projects `tailwind.config.js` to prevent usage)
- **Typography**: Always use the `Text` component for text styling instead of Tailwind's typography classes
- **Colors**: Use only design token generated color classes (e.g., `bg-default`, `text-error-default`)
- **Example**:

**React Web:**
```tsx
// ❌ Don't - Using default Tailwind colors, arbitrary color values or direct typography
<div className="bg-blue-500 text-white p-4 text-lg font-bold">
  <h1 className="text-2xl">Title</h1>
  <p className="text-gray-600">Content</p>
</div>

// ✅ Do - Using design token colors and Text component with enums
<Box backgroundColor="primary-default" padding={4}>
  <Text variant={TextVariant.HeadingLg} color={TextColor.PrimaryInverse}>
    Title
  </Text>
  <Text variant={TextVariant.BodyMd} color={TextColor.Alternative}>
    Content
  </Text>
</Box>
```

**React Native:**
```tsx
// ❌ Don't - Using StyleSheet with hardcoded values
const styles = StyleSheet.create({
  container: {
    backgroundColor: '#3B82F6',
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: 'white',
  },
  content: {
    fontSize: 16,
    color: '#6B7280',
  },
});

<View style={styles.container}>
  <Text style={styles.title}>Title</Text>
  <Text style={styles.content}>Content</Text>
</View>

// ✅ Do - Using design token colors and Text component with enums
<Box backgroundColor="primary-default" padding={4}>
  <Text variant={TextVariant.HeadingLg} color={TextColor.PrimaryInverse}>
    Title
  </Text>
  <Text variant={TextVariant.BodyMd} color={TextColor.Alternative}>
    Content
  </Text>
</Box>
```

## 💻 Platform-Specific Guidelines

### 3. React Web Components

- **Use Tailwind Utilities**: Leverage Tailwind's utility classes for styling via `className`
- **CVA Integration**: Coming soon - We are looking at implementing Class Variance Authority (CVA) for managing component variants. See [GitHub Issue #282](https://github.com/MetaMask/metamask-design-system/issues/282)
- **Example**:

```tsx
// ❌ Don't
const styles = {
  backgroundColor: '#F2F4F6',
  padding: '16px',
};

<div style={styles}>Content</div>

// ✅ Do - Using Box component with className for additional styling
<Box className="bg-alternative p-4">
  Content
</Box>

// ✅ Also acceptable - Using className when no prop exists
<Box className="bg-alternative p-4 my-2">
  Content with margin
</Box>
```

### 4. React Native Components

- **Design System TWRNC Preset**: Use `useTailwind` hook from `@metamask/design-system-twrnc-preset` instead of importing `twrnc` directly
- **Theme Integration**: The preset automatically handles light/dark theme switching and design token integration
- **Consistent API**: Maintain consistent class names between web and native
- **Example**:

```tsx
// ❌ Don't - Direct twrnc import
import tw from 'twrnc';

const styles = StyleSheet.create({
  container: {
    backgroundColor: colors.background.default,
    padding: 16,
  },
});

// ❌ Don't - Raw twrnc without design system integration
const styles = tw`bg-default p-4`;

// ✅ Do - Use design system preset with theme support
import { useTailwind } from '@metamask/design-system-twrnc-preset';

const MyComponent = () => {
  const tw = useTailwind();
  
  return (
    <View style={tw`bg-default p-4`}>
      Content
    </View>
  );
};

// ✅ Do - Using Box component with twClassName for additional styling
<Box twClassName="bg-alternative p-4">
  Content
</Box>

// ✅ Do - Using Pressable with tw function for interactive states
<Pressable
  style={({ pressed }) =>
    tw.style(
      'w-full flex-row items-center justify-between px-4 py-2',
      pressed && 'bg-pressed',
    )
  }
>
  Interactive Content
</Pressable>
```

## 🎨 Style Guidelines

### 5. Platform-Specific Styling Patterns

**React Web - className usage:**
```tsx
// Layout and spacing
<Box className="flex flex-row items-center gap-2 p-4">
  <Button>Buy/Sell</Button>
</Box>

// Interactive states (web supports hover/active)
<Box className="cursor-pointer hover:bg-hover active:bg-pressed">
  Clickable content
</Box>
```

**React Native - twClassName and tw usage:**
```tsx
// Layout and spacing with twClassName
<Box twClassName="flex flex-row items-center gap-2 p-4">
  <Button>Buy/Sell</Button>
</Box>

// Interactive states with tw function
<Pressable
  style={({ pressed }) =>
    tw.style(
      'w-full flex-row items-center justify-between px-4 py-2',
      pressed && 'bg-pressed',
    )
  }
>
  Interactive content
</Pressable>

// Direct tw usage for simple styling
<View style={tw`bg-default p-4 rounded-lg`}>
  Content
</View>
```

## 🛠️ Developer Tools & Configuration

### 6. IDE Integration

- **Enable Tailwind IntelliSense**: Use VSCode with Tailwind CSS IntelliSense plugin
- **Configure Workspace**: Follow `.vscode/settings.json` configuration:
  - Use experimental config file setup for monorepo
  - Enable string suggestions
  - Support custom functions: `tw`, `twClassName`, `twMerge`

### 7. Code Formatting

- **Prettier Integration**: Use Prettier with tailwind plugin
- **Consistent Ordering**: Maintain consistent class ordering through prettier-plugin-tailwindcss
- **Multiple Configs**: Respect the different Tailwind configs for React and React Native

## ⚠️ Common Pitfalls

### 8. Anti-patterns to Avoid

- **No Arbitrary Values**: Don't use `[]` syntax for arbitrary values unless absolutely necessary
- **No Direct Styles**: Avoid inline `style` objects
- **No @apply**: Don't use `@apply` in CSS files
- **No Style Mixing**: Don't mix Tailwind with other styling approaches like inline styles in the same component. However, combining component props with Tailwind classes via `className`/`twClassName`/`tw` is acceptable when no equivalent prop exists
- **No Default Colors**: Never use Tailwind's default color palette
- **No Direct Typography**: Never use typography classes directly

**Platform-specific examples:**

**React Web:**
```tsx
// ❌ Don't - Mixing inline styles with Tailwind classes
<div 
  className="bg-default p-4" 
  style={{ marginTop: '16px' }}
>
  Content
</div>

// ✅ Do - Using components with consistent patterns
<Box backgroundColor="default" padding={4} marginTop={4}>
  Content
</Box>

// ✅ Also acceptable - Combining props with className when no prop exists
<Box backgroundColor="default" padding={4} className="my-2">
  Content with margin
</Box>
```

**React Native:**
```tsx
// ❌ Don't - Mixing StyleSheet with twrnc
<View 
  style={[tw`bg-default p-4`, { marginTop: 16 }]}
>
  Content
</View>

// ✅ Do - Using components with consistent patterns
<Box backgroundColor="default" padding={4} marginTop={4}>
  Content
</Box>

// ✅ Also acceptable - Combining props with twClassName when no prop exists
<Box backgroundColor="default" padding={4} twClassName="my-2">
  Content with margin
</Box>
```

---

> This guide is a living document and will be updated as our design system evolves. For questions or suggestions, please reach out to the design system team. 