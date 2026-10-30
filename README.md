# expo-dev-menu without expo-dev-launcher: the Android menu never draws

A blank SDK 57 app (`npx create-expo-app@latest --template blank@sdk-57`) with only two changes:

- `expo-dev-menu@57.0.20` installed directly, and `expo-manifests`, which `expo-dev-menu`'s
  Android build needs but does not declare;
- `"plugins": ["expo-dev-menu"]` in `app.json`.

No `expo-dev-client`, so no `expo-dev-launcher`.

## Reproduce

```bash
npm install
npx expo run:android
```

Open the dev menu with `adb shell input keyevent 82`, or tap the floating tools button.

**Expected:** the dev menu sheet opens.
**Actual:** the tools button hides, as if the menu opened, but no sheet draws.

## The one-line change that fixes it

In `node_modules/expo-dev-menu/android/src/debug/java/expo/modules/devmenu/DevMenuPackage.kt`, add
`import com.facebook.react.ReactApplication` and replace

```kotlin
reactHostHolder = delegate?.reactHost.weak()
```

with

```kotlin
reactHostHolder = (delegate?.reactHost ?: (activity?.application as? ReactApplication)?.reactHost).weak()
```

then run `npx expo run:android` again: the menu opens.
