# Tailwind CSS Guidelines

Tailwind CSS conventions and best practices for using Tailwind CSS effectively and consistently across our engineering organization.

## Core Principles

### Component-First Approach

Always prefer components over raw JSX elements with Tailwind classes. Use component props to control variants and styles, but use `className`/`twClassName`/`tw` when no equivalent prop exists.

**React:**

❌ **Avoid**

```tsx
<div className="bg-default p-4 rounded-lg">Content</div>
```

✅ **Recommended**

```tsx
import {
  Box,
  BoxBackgroundColor,
  BoxBorderRadius,
} from '@metamask/design-system-react';

<Box
  backgroundColor={BoxBackgroundColor.BackgroundDefault}
  padding={4}
  borderRadius={BoxBorderRadius.Lg}
>
  Content
</Box>;
```

**React Native:**

❌ **Avoid**

```tsx
<View style={tw`bg-default p-4 rounded-lg`}>Content</View>
```

✅ **Recommended**

```tsx
import {
  Box,
  BoxBackgroundColor,
  BoxBorderRadius,
} from '@metamask/design-system-react-native';

<Box
  backgroundColor={BoxBackgroundColor.BackgroundDefault}
  padding={4}
  borderRadius={BoxBorderRadius.Lg}
>
  Content
</Box>;
```

### Color and Typography

Use only design token based classes that are generated from our design system. Never use Tailwind's default color palette, arbitrary colors like hex or rgb values, or font sizes. Always use the `Text` component for text styling instead of Tailwind's typography classes.

**React:**

❌ **Avoid**

```tsx
<div className="bg-blue-500 text-[#FFFFFF] p-4 text-lg font-bold">
  <h1 className="text-2xl">Title</h1>
</div>
<p className="text-gray-600">Content</p>
```

✅ **Recommended**

```tsx
import { Box, BoxBackgroundColor, BoxBorderRadius, Text, TextVariant, TextColor } from '@metamask/design-system-react';

<Box backgroundColor={BoxBackgroundColor.PrimaryDefault} padding={4}>
  <Text variant={TextVariant.HeadingLg} color={TextColor.PrimaryInverse}>
    Title
  </Text>
</Box>
<Text variant={TextVariant.BodyMd} color={TextColor.TextAlternative}>
  Content
</Text>
```

**React Native:**

❌ **Avoid**

```tsx
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
});

<View style={styles.container}>
  <Text style={styles.title}>Title</Text>
</View>;
```

✅ **Recommended**

```tsx
import { Box, BoxBackgroundColor, BoxBorderRadius, Text, TextVariant, TextColor } from '@metamask/design-system-react-native';

<Box backgroundColor={BoxBackgroundColor.PrimaryDefault} padding={4}>
  <Text variant={TextVariant.HeadingLg} color={TextColor.PrimaryInverse}>
    Title
  </Text>
</Box>
<Text variant={TextVariant.BodyMd} color={TextColor.TextAlternative}>
  Content
</Text>
```

## Platform-Specific Guidelines

### React

Leverage Tailwind's utility classes for styling via `className`.

❌ **Avoid**

```tsx
const styles = {
  backgroundColor: '#F2F4F6',
  padding: '16px',
};

<div style={styles}>Content</div>;
```

✅ **Recommended**

```tsx
import { Box, BoxBackgroundColor } from '@metamask/design-system-react';

<Box
  tabIndex={0}
  backgroundColor={BoxBackgroundColor.BackgroundDefault}
  className="hover:bg-hover active:bg-pressed" // When no prop exists
>
  Content
</Box>;
```

### React Native

Use `useTailwind` hook from `@metamask/design-system-twrnc-preset` instead of importing `twrnc` directly. The preset automatically handles light/dark theme switching and design token integration.

❌ **Avoid**

```tsx
import tw from 'twrnc';

const styles = StyleSheet.create({
  container: {
    backgroundColor: colors.background.default,
    padding: 16,
  },
});

// Direct twrnc without design system integration
const styles = tw`bg-default p-4`;
```

✅ **Recommended**

```tsx
import { useTailwind } from '@metamask/design-system-twrnc-preset';

const MyComponent = () => {
  const tw = useTailwind();

  return (
    <Pressable style={tw`bg-default p-4`}>
      Content
    </Pressable>
  );
};

// Using Box component with twClassName
<Box twClassName="overflow-hidden">
  Content
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
  Interactive Content
</Pressable>
```

## Style Guidelines

### Platform-Specific Styling Patterns

**React - className usage:**

```tsx
import { ButtonBase, Icon, IconName } from '@metamask/design-system-react';

// Use the className prop to override existing tailwind classnames when necessary
// Example overriding the default size/shape, layout and interactive states
<ButtonBase className="h-auto flex-1 flex-col justify-center rounded-lg bg-muted py-4 hover:bg-muted-hover active:bg-muted-pressed">
  <Icon name={IconName.Bank} className="mb-2" />
  Buy/Sell
</ButtonBase>;
```

**React Native - twClassName and tw usage:**

```tsx
import { ButtonBase, Icon, IconName, Box, BoxFlexDirection, BoxAlignItems, BoxJustifyContent, Text, FontWeight } from '@metamask/design-system-react-native';

// Custom overrides with twClassName
<ButtonBase twClassName="h-20 flex-1 rounded-lg bg-muted px-0 py-4">
  <Box
    flexDirection={BoxFlexDirection.Column}
    alignItems={BoxAlignItems.Center}
    justifyContent={BoxJustifyContent.Center}
  >
    <Icon name={IconName.Bank} />
    <Text fontWeight={FontWeight.Medium}>Buy/Sell</Text>
  </Box>
</ButtonBase>

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
<Scrollable style={tw`bg-default p-4 rounded-lg`}>
  Content
</Scrollable>
```

## Developer Tools and Configuration

### IDE Integration

- **Enable Tailwind IntelliSense**: Use VSCode with [Tailwind CSS IntelliSense](https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss) plugin
- **Configure Workspace**: Follow `.vscode/settings.json` configuration:
  - Enable string suggestions
  - Support custom functions: `tw`, `twClassName`, `twMerge`

Example [.vscode/settings.json](https://github.com/MetaMask/metamask-design-system/blob/main/.vscode/settings.json)

### Code Formatting

- **ESLint Integration**: Use the [tailwind eslint plugin](https://github.com/francoismassart/eslint-plugin-tailwindcss)
- **Consistent Ordering**: Maintain consistent class ordering through prettier-plugin-tailwindcss
- **Multiple Configs**: Respect the different Tailwind configs for React and React Native

Example [eslint.config.mjs](https://github.com/MetaMask/metamask-design-system/blob/main/eslint.config.mjs)

## Common Pitfalls

### Anti-patterns to Avoid

- **No Arbitrary Values**: Don't use `[]` syntax for arbitrary values unless absolutely necessary

- **No Direct Styles**: Avoid inline `style` objects unless necessary for dynamic values or cases where Tailwind cannot achieve the desired styling (e.g., custom animations or dynamic values like `style={{ marginTop: dynamicValue }}`)
- **No @apply**: Don't use `@apply` in CSS files
- **Avoid Style Mixing**: Try to avoid mixing Tailwind with other styling approaches like inline styles in the same component when not necessary. Style mixing may be necessary for custom animations or dynamic values that can't be achieved with Tailwind alone. However, combining component props with Tailwind classes via `className`/`twClassName`/`tw` is acceptable when no equivalent prop exists

**React:**

❌ **Avoid**

```tsx
// Arbitrary values
<button className="m-[16px] bg-[#FFFFFF] w-[100%] h-[100px]">Confirm</button>

// Unnecessary inline styles
<button style={{ marginTop: '16px' }}>Confirm</button>
```

```css
/* Using @apply */
.btn {
  @apply px-5 py-2 rounded-full cursor-pointer border disabled:cursor-auto;
}
```

```tsx
// Style mixing
<div className="bg-default p-4" style={{ marginTop: '16px' }}>
  Content
</div>
```

✅ **Recommended**

```tsx
// Use classes provided by Tailwind config and necessary arbitrary value
<button className="m-4 bg-default w-full h-[100px]">Content</button>;

// Necessary inline styles only
<button style={{ marginTop: dynamicValue }}>Confirm</button>;

// Create components instead of using @apply
import { twMerge } from '@metamask/design-system-react';

const MyButton = ({ className }) => (
  <button
    className={twMerge(
      'px-5 py-2 rounded-full cursor-pointer border disabled:cursor-auto',
      className,
    )}
  >
    Confirm
  </button>
);
```

**React Native:**

❌ **Avoid**

```tsx
// Style mixing
<Scrollable style={[tw`bg-default p-4`, { marginTop: 16 }]}>Content</Scrollable>

// Arbitrary values
<Pressable style={[tw`m-[16px] bg-[#FFFFFF] w-[100%] h-[100px]`]}>Confirm</Pressable>
```

✅ **Recommended**

```tsx
// Use classes provided by Tailwind config
<Scrollable style={tw`bg-default p-4 mt-4`}>Content</Scrollable>

// Use classes provided by Tailwind config and necessary arbitrary value
<Pressable style={tw`m-4 bg-default w-full h-[100px]`} >Confirm</Pressable>
```

---

> This guide is a living document and will be updated as our design system evolves. For questions or suggestions, please reach out to the [Design System Engineers](https://github.com/orgs/MetaMask/teams/design-system-engineers).
