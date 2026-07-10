# react-native-overlay-flow

Headless overlay orchestration for React Native.

`react-native-overlay-flow` helps you manage modals, bottom sheets, loaders, confirmations, toasts, and custom overlays from one place — without imposing any UI.

You bring your own components.
The library only coordinates **what appears**, **when it appears**, **how it stacks**, and **how it closes**.

---

## Why?

React Native apps often start with simple local states:

```tsx
const [isLoading, setIsLoading] = useState(false);
const [isSuccessOpen, setIsSuccessOpen] = useState(false);
const [isConfirmOpen, setIsConfirmOpen] = useState(false);
```

That works for small screens.

But in real apps, overlays quickly become harder to coordinate:

- a loader should be replaced by a success or error modal;
- a confirmation modal should return a result;
- a bottom sheet should open from business logic;
- a reward modal should wait until another modal closes;
- one overlay should appear above another;
- close animations should finish before unmounting;
- Android back should close the right overlay.

`react-native-overlay-flow` gives you a single headless layer to orchestrate those flows.

---

## Features

- Headless by design
- Bring your own UI
- Type-safe overlay registry
- Stack overlays above each other
- Queue overlays to show later
- Replace the current overlay
- Async result support with `ask()`
- Close lifecycle for animations
- Works with modals, bottom sheets, loaders, toasts, and custom overlays

---

## Installation

```sh
npm install react-native-overlay-flow
```

or:

```sh
yarn add react-native-overlay-flow
```

or:

```sh
pnpm add react-native-overlay-flow
```

---

## Quick start

### 1. Define your overlay types

```tsx
type AppOverlays = {
  'confirm-delete': {
    payload: {
      title: string;
      description?: string;
    };
    result: boolean;
  };

  'payment-success': {
    payload: {
      amount: number;
      transactionId: string;
    };
    result: void;
  };

  'global-loader': {
    payload: {
      message?: string;
    };
    result: void;
  };
};
```

---

### 2. Create your overlay components

```tsx
import { View, Text, Button } from 'react-native';
import type { OverlayComponentProps } from 'react-native-overlay-flow';

type Props = OverlayComponentProps<{
  payload: {
    title: string;
    description?: string;
  };
  result: boolean;
}>;

export function ConfirmDeleteOverlay({ payload, resolve }: Props) {
  return (
    <View>
      <Text>{payload.title}</Text>

      {payload.description ? <Text>{payload.description}</Text> : null}

      <Button title="Cancel" onPress={() => resolve(false)} />
      <Button title="Delete" onPress={() => resolve(true)} />
    </View>
  );
}
```

---

### 3. Register your overlays

```tsx
import { createOverlayRegistry } from 'react-native-overlay-flow';

import { ConfirmDeleteOverlay } from './ConfirmDeleteOverlay';
import { PaymentSuccessOverlay } from './PaymentSuccessOverlay';
import { GlobalLoaderOverlay } from './GlobalLoaderOverlay';

export const overlayRegistry = createOverlayRegistry<AppOverlays>({
  'confirm-delete': {
    component: ConfirmDeleteOverlay,
    defaultOptions: {
      closeOnBack: true,
      dismissible: false,
      exitDuration: 250,
    },
  },

  'payment-success': {
    component: PaymentSuccessOverlay,
    defaultOptions: {
      closeOnBack: true,
      dismissible: true,
      exitDuration: 250,
    },
  },

  'global-loader': {
    component: GlobalLoaderOverlay,
    defaultOptions: {
      closeOnBack: false,
      dismissible: false,
      exitDuration: 150,
    },
  },
});
```

---

### 4. Add the provider and host

```tsx
import { OverlayProvider, OverlayHost } from 'react-native-overlay-flow';
import { overlayRegistry } from './overlays';

export function App() {
  return (
    <OverlayProvider registry={overlayRegistry}>
      <AppNavigator />
      <OverlayHost />
    </OverlayProvider>
  );
}
```

---

### 5. Open overlays from anywhere

```tsx
import { useOverlay } from 'react-native-overlay-flow';

function PaymentScreen() {
  const overlay = useOverlay<AppOverlays>();

  function showSuccess() {
    overlay.open('payment-success', {
      amount: 12000,
      transactionId: 'TX-123',
    });
  }

  return <Button title="Show success" onPress={showSuccess} />;
}
```

---

## Full guide

For `ask()`, `replace()`, `enqueue()`, the close lifecycle, bottom sheets and
custom overlays, and the full API reference, see the [Guide](./GUIDE.md).

---

## Philosophy

This is not a modal library.
This is not a bottom sheet library.
This is not a toast library.

It is an orchestration layer.

Bring your own UI.
Bring your own modal.
Bring your own bottom sheet.
Bring your own toast.

`react-native-overlay-flow` only coordinates the flow.

---

## When should I use this?

Use local state for simple screen-only UI.

```tsx
const [isOpen, setIsOpen] = useState(false);
```

Use `react-native-overlay-flow` when overlays become global, stacked, queued, async, or shared across features.

Good use cases:

- confirmation modals;
- global loaders;
- payment success or error modals;
- session expired modals;
- bottom sheets opened from business logic;
- reward or bonus modals;
- rate app prompts;
- multi-step overlay flows.

---

## Roadmap

### Scaffolding

- [x] Project structure & tooling: bob, eslint, jest, turbo, example app
- [x] Public API surface defined: `src/index.tsx` exports
- [x] Documentation & usage guide
- [ ] Remove leftover template boilerplate: `src/multiply.tsx`

### Core

- [ ] Core types: `OverlayComponentProps`, `OverlayOptions`, `OverlayStatus`, registry types
- [ ] `createOverlayRegistry()`
- [ ] `createOverlayId()`
- [ ] `OverlayStateProvider`
- [ ] `OverlayActionProvider`
- [ ] `OverlayProvider`
- [ ] `OverlayHost`
- [ ] `OverlayLayer`
- [ ] `useOverlay()`
- [ ] `useOverlayEntry()`

### Behavior

- [ ] Core overlay stack: `open`, `close`, stacking
- [ ] Queue support: `enqueue`
- [ ] Replace support: `replace`
- [ ] Async result support: `ask`, `resolve`, `reject`
- [ ] Close lifecycle: `opening`, `open`, `closing`, `exitDuration`, `onExitComplete`
- [ ] Android back handling: `closeOnBack`

### Quality and extras

- [ ] Test suite
- [ ] Example app wired to the library
- [ ] Optional Gorhom Bottom Sheet adapter
- [ ] Optional toast adapter
- [ ] Devtools/debug panel

---

## License

MIT
