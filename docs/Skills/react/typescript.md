# TypeScript Integration

## Setup
```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "strict": true,
    "lib": ["DOM", "ES2022"],
    "module": "ESNext",
    "moduleResolution": "bundler"
  }
}
```

## Essential types (React 19)
```tsx
import {
  type ReactNode,
  type PropsWithChildren,
  type ComponentProps,
} from 'react';

// Props — prefer plain functions over FC<> annotations
type ButtonProps = PropsWithChildren<{
  variant?: 'primary' | 'ghost';
  onClick?: () => void;
}>;

function Button({ children, variant = 'primary' }: ButtonProps) {
  return <button className={variant}>{children}</button>;
}

// Extract props from a host element
type InputProps = ComponentProps<'input'>;

// React 19: ref is just a prop — no forwardRef
type SearchProps = ComponentProps<'input'> & { ref?: React.Ref<HTMLInputElement> };
```

**Rule**: Prefer plain function components with typed props over `React.FC` — avoids implicit `children`, plays better with generics and inference.

## React 19 type changes
- `forwardRef` deprecated — `ref` is a normal prop
- `element.ref` deprecated — read `element.props.ref`
- `defaultProps` removed from function-component types too — use default parameters
- `ReactNode` includes promises now (`Promise<ReactNode>`), matching async Server Components and `use()`

## Generics in components
```tsx
function List<T>({ items, render }: {
  items: T[];
  render: (item: T) => ReactNode;
}) {
  return <ul>{items.map(render)}</ul>;
}
```

## Migrating types to React 19
```bash
npx types-react-codemod@latest preset-19 ./src
```

→ Sources: [react.dev/learn/typescript](https://react.dev/learn/typescript), [types-react-codemod](https://github.com/eps1lon/types-react-codemod)
