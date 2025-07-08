# Tailwind CSS Guidelines

Tailwind CSS conventions and best practices for using Tailwind CSS effectively and consistently across our engineering organization.

## Core Principles

### Component-First Approach

Always prefer components over raw JSX elements with Tailwind classes. Use component props to control variants and styles, but use `className`/`twClassname`/`tw` when no equivalent prop exists.

**React:**

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

❌ **Avoid**

```tsx
<div className="bg-default p-4 rounded-lg">Content</div>
```

**React Native:**

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

❌ **Avoid**

```tsx
<View style={tw`bg-default p-4 rounded-lg`}>Content</View>
```

### Color and Typography

Use only design token based classes that are generated from our design system. Never use Tailwind's default color palette, arbitrary colors like hex or rgb values or font sizes. Always use the `Text` component for text styling instead of Tailwind's typography classes.

**React:**

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

❌ **Avoid**

```tsx
<div className="bg-blue-500 text-white p-4 text-lg font-bold">
  <h1 className="text-2xl">Title</h1>
</div>
<p className="text-gray-600">Content</p>
```

**React Native:**

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

## Platform-Specific Guidelines

### React

Leverage Tailwind's utility classes for styling via `className`.

✅ **Recommended**

```tsx
import { Box, BoxBackgroundColor } from '@metamask/design-system-react';

// When no prop exists
<Box
  tabIndex={0}
  backgroundColor={BoxBackgroundColor.BackgroundDefault}
  className="hover:bg-hover active:bg-pressed"
>
  Content
</Box>;
```

❌ **Avoid**

```tsx
const styles = {
  backgroundColor: '#F2F4F6',
  padding: '16px',
};

<div style={styles}>Content</div>;
```

### React Native Components

Use `useTailwind` hook from `@metamask/design-system-twrnc-preset` instead of importing `twrnc` directly. The preset automatically handles light/dark theme switching and design token integration.

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
<Box twClassName="h-[100px]">
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

## Style Guidelines

### Platform-Specific Styling Patterns

**React - className usage:**

```tsx
import { ButtonBase, Icon, IconName } from '@metamask/design-system-react';

// Layout and spacing
// Interactive states (react supports hover/active)
<ButtonBase className="h-auto flex-1 flex-col justify-center rounded-lg bg-muted py-4 hover:bg-muted-hover active:bg-muted-pressed">
  <Icon name={IconName.Bank} className="mb-2" />
  Buy/Sell
</ButtonBase>;
```

**React Native - twClassName and tw usage:**

```tsx
import { ButtonBase, Icon, IconName, Box, BoxFlexDirection, BoxAlignItems, BoxJustifyContent, Text, FontWeight } from '@metamask/design-system-react';

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

- **Eslint Integration**: Use the [tailwind eslint plugin](https://github.com/francoismassart/eslint-plugin-tailwindcss)
- **Consistent Ordering**: Maintain consistent class ordering through prettier-plugin-tailwindcss
- **Multiple Configs**: Respect the different Tailwind configs for React and React Native

Example [eslint.config.mjs](https://github.com/MetaMask/metamask-design-system/blob/main/eslint.config.mjs)

## Common Pitfalls

### Anti-patterns to Avoid

- **No Arbitrary Values**: Don't use `[]` syntax for arbitrary values unless absolutely necessary
- **No Direct Styles**: Avoid inline `style` objects
- **No @apply**: Don't use `@apply` in CSS files
- **Avoid Style Mixing**: Try to avoid mixing Tailwind with other styling approaches like inline styles in the same component when not necessary. Style mixing may be necessary for custom animations or dynamic values that can't be achieved with Tailwind alone. However, combining component props with Tailwind classes via `className`/`twClassName`/`tw` is acceptable when no equivalent prop exists
- **No Default Colors**: Never use Tailwind's default color palette
- **No Direct Typography**: Never use typography classes directly

**React:**

❌ **Avoid**

```tsx
<div className="bg-default p-4" style={{ marginTop: '16px' }}>
  Content
</div>
```

✅ **Recommended**

```tsx
<Box backgroundColor={BoxBackgroundColor.BackgroundDefault} padding={4} marginTop={4}>
  Content
</Box>

// When no prop exists
<Box backgroundColor={BoxBackgroundColor.BackgroundDefault} padding={4} className="hover:bg-hover">
  Content with margin
</Box>
```

**React Native:**

❌ **Avoid**

```tsx
<View style={[tw`bg-default p-4`, { marginTop: 16 }]}>Content</View>
```

✅ **Recommended**

```tsx
<Box backgroundColor={BoxBackgroundColor.BackgroundDefault} padding={4} marginTop={4}>
  Content
</Box>

// When no prop exists
<Box backgroundColor={BoxBackgroundColor.BackgroundDefault} padding={4} twClassName="my-2">
  Content with margin
</Box>
```

---

> This guide is a living document and will be updated as our design system evolves. For questions or suggestions, please reach out to the [Design System Engineers](https://github.com/orgs/MetaMask/teams/design-system-engineers).
