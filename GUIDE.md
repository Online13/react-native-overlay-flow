# Guide

Full usage guide and API reference for `react-native-overlay-flow`.

For installation and a quick start, see the [README](./README.md).

## Contents

- [Opening overlays](#opening-overlays)
- [Asking for a result](#asking-for-a-result)
- [Replacing the current overlay](#replacing-the-current-overlay)
- [Queueing an overlay](#queueing-an-overlay)
- [Closing overlays](#closing-overlays)
- [Close lifecycle](#close-lifecycle)
- [Bottom sheets and custom overlays](#bottom-sheets-and-custom-overlays)
- [API reference](#api-reference)
  - [Components](#components)
  - [Hook](#hook)
  - [Methods](#methods)
  - [Overlay options](#overlay-options)

---

## Opening overlays

```tsx
overlay.open('payment-success', {
  amount: 12000,
  transactionId: 'TX-123',
});
```

If another overlay is already visible, the new one appears above it.

```tsx
overlay.open('payment-success', {
  amount: 12000,
  transactionId: 'TX-123',
});

overlay.open('confirm-delete', {
  title: 'Delete transaction?',
});
```

---

## Asking for a result

Use `ask()` when an overlay should return a value.

```tsx
const confirmed = await overlay.ask('confirm-delete', {
  title: 'Delete this item?',
  description: 'This action cannot be undone.',
});

if (confirmed) {
  await deleteItem();
}
```

Inside the overlay component, call `resolve()`.

```tsx
function ConfirmDeleteOverlay({ payload, resolve }: Props) {
  return (
    <ConfirmModal
      title={payload.title}
      description={payload.description}
      onCancel={() => resolve(false)}
      onConfirm={() => resolve(true)}
    />
  );
}
```

---

## Replacing the current overlay

Use `replace()` when the current overlay should become another one.

```tsx
overlay.open('global-loader', {
  message: 'Processing payment...',
});

try {
  const result = await paymentApi.pay();

  overlay.replace('payment-success', {
    amount: result.amount,
    transactionId: result.transactionId,
  });
} catch {
  overlay.replace('payment-error', {
    message: 'Payment failed',
  });
}
```

Useful for flows like:

```txt
loader → success
loader → error
confirm → processing
```

---

## Queueing an overlay

Use `enqueue()` when an overlay should wait until the current overlays are closed.

```tsx
overlay.open('payment-success', {
  amount: 12000,
  transactionId: 'TX-123',
});

overlay.enqueue('rate-app');
```

Result:

```txt
1. payment-success appears
2. user closes payment-success
3. rate-app appears
```

---

## Closing overlays

Close the top overlay:

```tsx
overlay.closeTop();
```

Close a specific overlay by id:

```tsx
const id = overlay.open('payment-success', {
  amount: 12000,
  transactionId: 'TX-123',
});

overlay.close(id);
```

Close all overlays:

```tsx
overlay.closeAll();
```

---

## Close lifecycle

Overlays are not removed immediately when they close.

Instead, they receive a `status`:

```tsx
type OverlayStatus = 'opening' | 'open' | 'closing';
```

When `status` becomes `"closing"`, the overlay can run its exit animation.

After the animation finishes, call:

```tsx
onExitComplete();
```

Example:

```tsx
function AnimatedOverlay({
  status,
  close,
  onExitComplete,
}: OverlayComponentProps<any>) {
  return (
    <MyAnimatedModal
      visible={status !== 'closing'}
      onClose={close}
      onExitComplete={onExitComplete}
    />
  );
}
```

If `onExitComplete()` is not called, the overlay can be removed automatically after `exitDuration`.

---

## Bottom sheets and custom overlays

The library does not provide a bottom sheet component.

Use your own bottom sheet library, such as `@gorhom/bottom-sheet`, and register the sheet as an overlay.

```tsx
import { useEffect, useRef } from 'react';
import { BottomSheetModal } from '@gorhom/bottom-sheet';
import type { OverlayComponentProps } from 'react-native-overlay-flow';

export function FilterSheetOverlay({
  payload,
  status,
  close,
  onExitComplete,
}: OverlayComponentProps<any>) {
  const ref = useRef<BottomSheetModal>(null);

  useEffect(() => {
    ref.current?.present();
  }, []);

  useEffect(() => {
    if (status === 'closing') {
      ref.current?.dismiss();
    }
  }, [status]);

  return (
    <BottomSheetModal ref={ref} onDismiss={onExitComplete}>
      <FilterContent payload={payload} onApply={close} />
    </BottomSheetModal>
  );
}
```

Then open it normally:

```tsx
overlay.open('filter-sheet', {
  initialCategory: 'phones',
});
```

The same pattern applies to toasts, custom drawers, or any other overlay type — the library only needs a component to mount.

---

## API reference

### Components

```tsx
<OverlayProvider registry={overlayRegistry}>
  <App />
  <OverlayHost />
</OverlayProvider>
```

| Component          | Role                        |
| ------------------ | ---------------------------- |
| `OverlayProvider`  | Stores the overlay state.    |
| `OverlayHost`      | Renders the active overlays. |

### Hook

```tsx
const overlay = useOverlay<AppOverlays>();
```

### Methods

```tsx
overlay.open(name, payload?, options?);
overlay.ask(name, payload?, options?);
overlay.replace(name, payload?, options?);
overlay.enqueue(name, payload?, options?);

overlay.close(id);
overlay.closeTop();
overlay.closeAll();
```

### Overlay options

```tsx
type OverlayOptions = {
  dismissible?: boolean;
  closeOnBack?: boolean;
  exitDuration?: number;
};
```
