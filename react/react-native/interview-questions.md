# React Native Interview Questions & Answers

---

## 1. What is React Native?

React Native is an open-source framework developed by Facebook for building native mobile applications using JavaScript and React. Unlike hybrid frameworks that render in a WebView, React Native renders using actual native platform components (UIView on iOS, android.view on Android).

---

## 2. How is React Native different from React.js?

| Feature | React.js | React Native |
|---|---|---|
| Platform | Web (browser) | Mobile (iOS, Android) |
| Rendering | HTML DOM elements | Native platform components |
| Styling | CSS | StyleSheet (JavaScript objects, flexbox-based) |
| Navigation | React Router, etc. | React Navigation, Expo Router |
| Output | HTML/CSS/JS | Native iOS/Android views |
| Base components | `<div>`, `<span>`, `<p>` | `<View>`, `<Text>`, `<Image>` |

Core React concepts (components, state, props, hooks) remain the same.

---

## 3. What are the core components in React Native?

| Component | Description | Web Equivalent |
|---|---|---|
| `<View>` | Container for layout | `<div>` |
| `<Text>` | Display text | `<p>`, `<span>` |
| `<Image>` | Display images | `<img>` |
| `<TextInput>` | Text input field | `<input type="text">` |
| `<ScrollView>` | Scrollable container | `<div>` with overflow scroll |
| `<FlatList>` | Performant scrollable list | No direct equivalent |
| `<SectionList>` | Grouped list with headers | No direct equivalent |
| `<TouchableOpacity>` | Touchable with opacity feedback | `<button>` |
| `<Pressable>` | Flexible touch handler | `<button>` |
| `<Modal>` | Modal overlay | Modal dialog |
| `<ActivityIndicator>` | Loading spinner | Spinner |

---

## 4. How does React Native work under the hood?

React Native has three main threads:

1. **JS Thread** — runs the JavaScript code (business logic, React rendering)
2. **Main/UI Thread** — handles native UI rendering and user interactions
3. **Shadow Thread** — calculates layout using Yoga (a cross-platform layout engine based on flexbox)

Communication between JS and native was handled by the **Bridge** (JSON serialization over async messaging). The new architecture replaces this with **JSI (JavaScript Interface)** for synchronous, direct communication.

---

## 5. What is the New Architecture in React Native?

The New Architecture (introduced gradually from RN 0.68+) includes:

- **JSI (JavaScript Interface)** — allows JS to directly call native methods without the bridge, enabling synchronous calls and shared memory
- **Fabric** — new rendering system that replaces the old UI manager, enabling synchronous rendering and concurrent features
- **TurboModules** — lazy-loaded native modules that are initialized only when needed, improving startup time
- **Codegen** — generates native code from JavaScript type definitions for type-safe communication

---

## 6. What is the Bridge and why is it being replaced?

The Bridge was React Native's original communication layer between JS and native. It serialized data as JSON and passed messages asynchronously.

Problems with the Bridge:
- **Asynchronous only** — no synchronous calls, causing latency
- **Serialization overhead** — all data converted to/from JSON
- **Bottleneck** — single-threaded message queue could get congested
- **No direct memory sharing** — data had to be copied between realms

JSI replaces it with direct, synchronous method invocations and shared memory.

---

## 7. What is the difference between `ScrollView` and `FlatList`?

| Feature | ScrollView | FlatList |
|---|---|---|
| Rendering | Renders all children at once | Renders items lazily (windowed) |
| Performance | Poor with large lists | Optimized for large lists |
| Memory | Loads everything into memory | Only keeps visible items in memory |
| Use case | Small scrollable content | Long lists of data |
| Required props | `children` | `data` and `renderItem` |

Always use `FlatList` (or `SectionList`) for lists with more than a handful of items.

---

## 8. How do you optimize FlatList performance?

Key optimizations:

```jsx
<FlatList
  data={items}
  renderItem={renderItem}
  keyExtractor={item => item.id}
  getItemLayout={(data, index) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  })}
  maxToRenderPerBatch={10}
  windowSize={5}
  removeClippedSubviews={true}
  initialNumToRender={10}
/>
```

- **`keyExtractor`** — provide stable unique keys
- **`getItemLayout`** — skip layout measurement when item height is fixed
- **`maxToRenderPerBatch`** — control how many items render per batch
- **`windowSize`** — control the number of pages rendered above/below the visible area
- **`removeClippedSubviews`** — unmount components outside the viewport (Android)
- **`React.memo`** — memoize the `renderItem` component
- **Avoid inline functions** in `renderItem`

---

## 9. How does styling work in React Native?

React Native uses JavaScript objects for styling via `StyleSheet.create()`. It uses flexbox for layout (with `flexDirection` defaulting to `column` instead of `row`).

```jsx
import { StyleSheet, View, Text } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#333',
  },
});

const App = () => (
  <View style={styles.container}>
    <Text style={styles.title}>Hello</Text>
  </View>
);
```

Key differences from CSS:
- Properties are camelCase (`backgroundColor` not `background-color`)
- No units — values are density-independent pixels by default
- No cascading or inheritance (except some `Text` properties)
- No media queries (use `Dimensions` API or `useWindowDimensions`)
- `flexDirection` defaults to `column`

---

## 10. What is the difference between `Pressable` and `TouchableOpacity`?

`Pressable` is the newer, more flexible API (introduced in RN 0.63):

```jsx
// TouchableOpacity — reduces opacity on press
<TouchableOpacity onPress={handlePress} activeOpacity={0.7}>
  <Text>Press me</Text>
</TouchableOpacity>

// Pressable — fully customizable press feedback
<Pressable
  onPress={handlePress}
  onPressIn={handlePressIn}
  onPressOut={handlePressOut}
  onLongPress={handleLongPress}
  style={({ pressed }) => [
    styles.button,
    pressed && styles.buttonPressed,
  ]}
  hitSlop={10}
  android_ripple={{ color: 'rgba(0,0,0,0.1)' }}
>
  <Text>Press me</Text>
</Pressable>
```

`Pressable` is recommended for new code as it provides more control over press states, hit slop, and platform-specific feedback.

---

## 11. What is the difference between Expo and bare React Native (CLI)?

| Feature | Expo | Bare React Native (CLI) |
|---|---|---|
| Setup | `npx create-expo-app` | `npx react-native init` |
| Native code access | Limited (unless using dev builds) | Full access |
| Build process | Expo cloud builds (EAS) | Xcode / Android Studio |
| OTA updates | Built-in (EAS Update) | Manual (CodePush, etc.) |
| Native modules | Must be compatible with Expo | Any native module |
| Ease of use | Easier, more abstracted | More control, more setup |
| Ejecting | Can create dev builds or prebuild | N/A |

Expo is recommended for most projects. With Expo Dev Builds and Config Plugins, the gap has narrowed significantly.

---

## 12. What is Expo Application Services (EAS)?

EAS is Expo's cloud-based toolchain for building, submitting, and updating React Native apps:

- **EAS Build** — cloud builds for iOS and Android (no need for local Xcode/Android Studio)
- **EAS Submit** — automated app store submissions
- **EAS Update** — over-the-air JavaScript updates without app store review

---

## 13. What is the difference between `useState` and `useReducer` in React Native?

The usage is identical to React.js:

- **`useState`** — simple state with a value and setter
- **`useReducer`** — complex state with a reducer function and dispatch

Use `useReducer` when:
- State logic involves multiple sub-values
- Next state depends on previous state
- You want to centralize state update logic

---

## 14. How does navigation work in React Native?

React Native has no built-in navigation. The most popular library is **React Navigation**.

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

// Navigate
navigation.navigate('Details', { itemId: 42 });
```

Navigation types:
- **Stack Navigator** — screen-to-screen navigation with a back stack
- **Tab Navigator** — bottom or top tab navigation
- **Drawer Navigator** — side menu navigation
- **Native Stack Navigator** — uses native navigation primitives for better performance

**Expo Router** is a newer file-based routing alternative built on top of React Navigation.

---

## 15. How do you handle deep linking in React Native?

Deep linking allows opening specific screens via URLs.

With React Navigation:

```jsx
const linking = {
  prefixes: ['myapp://', 'https://myapp.com'],
  config: {
    screens: {
      Home: '',
      Profile: 'user/:id',
      Settings: 'settings',
    },
  },
};

<NavigationContainer linking={linking}>
  {/* navigators */}
</NavigationContainer>
```

For Expo, use `expo-linking` to handle URL schemes and universal links.

---

## 16. How do you handle platform-specific code?

**Using `Platform` module:**

```jsx
import { Platform } from 'react-native';

const styles = StyleSheet.create({
  container: {
    paddingTop: Platform.OS === 'ios' ? 44 : 0,
    ...Platform.select({
      ios: { shadowColor: '#000', shadowOpacity: 0.2 },
      android: { elevation: 4 },
    }),
  },
});
```

**Using platform-specific file extensions:**

```
Button.ios.js    // used on iOS
Button.android.js // used on Android
```

React Native automatically picks the correct file based on the platform.

---

## 17. What is the `Animated` API?

`Animated` is React Native's built-in animation library for creating performant animations.

```jsx
import { Animated } from 'react-native';

const fadeAnim = useRef(new Animated.Value(0)).current;

const fadeIn = () => {
  Animated.timing(fadeAnim, {
    toValue: 1,
    duration: 500,
    useNativeDriver: true,
  }).start();
};

return (
  <Animated.View style={{ opacity: fadeAnim }}>
    <Text>Fading in</Text>
  </Animated.View>
);
```

Animation types:
- **`Animated.timing`** — animate over time with easing
- **`Animated.spring`** — spring physics animation
- **`Animated.decay`** — deceleration animation

Composition:
- **`Animated.parallel`** — run animations simultaneously
- **`Animated.sequence`** — run animations one after another
- **`Animated.stagger`** — run animations with staggered start times

Always set `useNativeDriver: true` when animating transform and opacity for better performance.

---

## 18. What is `react-native-reanimated`?

`react-native-reanimated` is a more powerful animation library that runs animations entirely on the native UI thread, avoiding the JS thread bottleneck.

```jsx
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';

const offset = useSharedValue(0);

const animatedStyle = useAnimatedStyle(() => ({
  transform: [{ translateX: offset.value }],
}));

const handlePress = () => {
  offset.value = withSpring(200);
};

return <Animated.View style={[styles.box, animatedStyle]} />;
```

Key concepts:
- **Shared Values** — values shared between JS and native threads
- **Worklets** — small JS functions that run on the UI thread
- **`useAnimatedStyle`** — creates styles driven by shared values
- **`useAnimatedGestureHandler`** — integrates with `react-native-gesture-handler`

---

## 19. What is `react-native-gesture-handler`?

`react-native-gesture-handler` provides native-driven gesture management, replacing React Native's built-in gesture system for better performance.

Key gestures:
- `Tap` — single and double tap
- `Pan` — dragging
- `Pinch` — pinch-to-zoom
- `Rotation` — two-finger rotation
- `LongPress` — press and hold
- `Fling` — swipe gestures

```jsx
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

const pan = Gesture.Pan()
  .onUpdate((event) => {
    translateX.value = event.translationX;
    translateY.value = event.translationY;
  })
  .onEnd(() => {
    translateX.value = withSpring(0);
    translateY.value = withSpring(0);
  });

return (
  <GestureDetector gesture={pan}>
    <Animated.View style={animatedStyle} />
  </GestureDetector>
);
```

---

## 20. How do you handle data persistence in React Native?

| Method | Use Case |
|---|---|
| `AsyncStorage` | Simple key-value storage (similar to localStorage) |
| `MMKV` (`react-native-mmkv`) | High-performance key-value storage |
| `SQLite` (`expo-sqlite`) | Structured relational data |
| `Realm` | Object-oriented local database |
| `WatermelonDB` | Large-scale offline-first apps |
| `SecureStore` (`expo-secure-store`) | Sensitive data (tokens, passwords) |

```jsx
// AsyncStorage
import AsyncStorage from '@react-native-async-storage/async-storage';

await AsyncStorage.setItem('token', 'abc123');
const token = await AsyncStorage.getItem('token');

// MMKV (much faster)
import { MMKV } from 'react-native-mmkv';
const storage = new MMKV();
storage.set('token', 'abc123');
const token = storage.getString('token');
```

---

## 21. How do you make network requests in React Native?

React Native supports `fetch` and `XMLHttpRequest` out of the box.

```jsx
// Using fetch
const fetchUsers = async () => {
  try {
    const response = await fetch('https://api.example.com/users');
    const data = await response.json();
    setUsers(data);
  } catch (error) {
    console.error(error);
  }
};
```

Popular libraries:
- **Axios** — HTTP client with interceptors, request cancellation
- **TanStack Query (React Query)** — data fetching with caching, refetching, pagination
- **SWR** — stale-while-revalidate data fetching
- **Apollo Client** — for GraphQL APIs

---

## 22. How do you handle images in React Native?

```jsx
// Local image
<Image source={require('./assets/logo.png')} style={{ width: 100, height: 100 }} />

// Remote image (must specify dimensions)
<Image source={{ uri: 'https://example.com/photo.jpg' }} style={{ width: 200, height: 200 }} />

// Background image
<ImageBackground source={require('./bg.png')} style={styles.background}>
  <Text>Content over image</Text>
</ImageBackground>
```

For better performance with remote images, use `expo-image` or `react-native-fast-image`:
- Disk and memory caching
- Priority-based loading
- Placeholder and transition support
- Progressive loading

---

## 23. What is `SafeAreaView` and why is it needed?

`SafeAreaView` renders content within the safe area boundaries of a device, avoiding notches, status bars, home indicators, and rounded corners.

```jsx
import { SafeAreaView } from 'react-native-safe-area-context';

function App() {
  return (
    <SafeAreaProvider>
      <SafeAreaView style={{ flex: 1 }}>
        <Text>This avoids the notch</Text>
      </SafeAreaView>
    </SafeAreaProvider>
  );
}
```

Use `react-native-safe-area-context` instead of React Native's built-in `SafeAreaView`, as it works on both iOS and Android and provides the `useSafeAreaInsets` hook for fine-grained control.

---

## 24. How do you handle push notifications?

Popular libraries:
- **`expo-notifications`** — full notification support for Expo projects
- **`react-native-firebase` (FCM)** — Firebase Cloud Messaging
- **`@notifee/react-native`** — local notifications with advanced features

Basic setup with Expo:

```jsx
import * as Notifications from 'expo-notifications';

// Request permissions
const { status } = await Notifications.requestPermissionsAsync();

// Get push token
const token = (await Notifications.getExpoPushTokenAsync()).data;

// Handle received notifications
Notifications.addNotificationReceivedListener(notification => {
  console.log(notification);
});

// Schedule local notification
await Notifications.scheduleNotificationAsync({
  content: { title: 'Reminder', body: 'Check your tasks' },
  trigger: { seconds: 60 },
});
```

---

## 25. How do you handle permissions in React Native?

```jsx
// Expo
import * as Location from 'expo-location';

const { status } = await Location.requestForegroundPermissionsAsync();
if (status !== 'granted') {
  alert('Permission denied');
  return;
}
const location = await Location.getCurrentPositionAsync();

// Bare React Native
import { PermissionsAndroid, Platform } from 'react-native';

if (Platform.OS === 'android') {
  const granted = await PermissionsAndroid.request(
    PermissionsAndroid.PERMISSIONS.CAMERA,
    {
      title: 'Camera Permission',
      message: 'App needs camera access',
      buttonPositive: 'OK',
    }
  );
}
```

For bare React Native, `react-native-permissions` provides a unified cross-platform API.

---

## 26. What is the difference between `useWindowDimensions` and `Dimensions` API?

```jsx
// Dimensions (static, doesn't update on rotation)
import { Dimensions } from 'react-native';
const { width, height } = Dimensions.get('window');

// useWindowDimensions (reactive, updates on rotation/resize)
import { useWindowDimensions } from 'react-native';
const { width, height } = useWindowDimensions();
```

Prefer `useWindowDimensions` because it automatically updates when the screen dimensions change (rotation, split-screen, foldable devices).

---

## 27. How do you handle forms in React Native?

React Native has no built-in form management. Common approaches:

**Basic controlled inputs:**

```jsx
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');

<TextInput value={email} onChangeText={setEmail} placeholder="Email" />
<TextInput value={password} onChangeText={setPassword} secureTextEntry />
```

**Using libraries:**
- **React Hook Form** — performant form management with minimal re-renders
- **Formik** — form state management with Yup validation

```jsx
// React Hook Form
import { useForm, Controller } from 'react-hook-form';

const { control, handleSubmit, errors } = useForm();

<Controller
  control={control}
  name="email"
  rules={{ required: 'Email is required' }}
  render={({ field: { onChange, value } }) => (
    <TextInput value={value} onChangeText={onChange} />
  )}
/>
```

---

## 28. What is Hermes?

Hermes is a JavaScript engine optimized for React Native. It is the default engine since React Native 0.70.

Benefits:
- Faster app startup (bytecode precompilation)
- Lower memory usage
- Smaller app size
- Optimized for mobile devices

Hermes supports most ES2015+ features and has its own garbage collector designed for mobile.

---

## 29. How do you debug React Native apps?

Tools and techniques:

- **React Native DevTools** — official debugging tool (replaces Flipper)
- **Chrome DevTools** — remote JS debugging
- **React DevTools** — inspect component tree, props, and state
- **Console logging** — `console.log`, `console.warn`, `console.error`
- **LogBox** — in-app error/warning display
- **Network inspector** — inspect API calls
- **Xcode (iOS)** — native crash logs, Instruments profiler
- **Android Studio (Android)** — Logcat, Layout Inspector

```jsx
// Disable LogBox warnings in development
import { LogBox } from 'react-native';
LogBox.ignoreLogs(['Warning: ...']);
```

---

## 30. How do you handle environment variables in React Native?

**Expo:**

```bash
# .env
API_URL=https://api.example.com
```

Access via `expo-constants` or `app.config.js`:

```jsx
// app.config.js
export default {
  extra: {
    apiUrl: process.env.API_URL,
  },
};

// In your app
import Constants from 'expo-constants';
const apiUrl = Constants.expoConfig.extra.apiUrl;
```

**Bare React Native:**
Use `react-native-config`:

```jsx
import Config from 'react-native-config';
const apiUrl = Config.API_URL;
```

Never store secrets in environment variables on the client side — they are bundled into the app binary.

---

## 31. What are Native Modules?

Native Modules allow you to write native code (Swift/Objective-C for iOS, Kotlin/Java for Android) and expose it to JavaScript.

Use cases:
- Accessing platform APIs not available in React Native
- Performance-critical operations
- Integrating native SDKs

With the New Architecture, TurboModules replace traditional Native Modules for better performance and type safety.

---

## 32. What is CodePush?

CodePush (now App Center CodePush) allows you to push JavaScript and asset updates directly to users' devices without going through the app store review process.

Alternatives:
- **EAS Update** (Expo) — Expo's built-in OTA update system
- **`expo-updates`** — library for managing OTA updates

Limitations:
- Cannot update native code (only JS bundle and assets)
- Major version changes still require store submission

---

## 33. How does React Native handle keyboard interactions?

```jsx
import { KeyboardAvoidingView, Platform, Keyboard } from 'react-native';

// Automatically adjust view when keyboard appears
<KeyboardAvoidingView
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  style={{ flex: 1 }}
>
  <TextInput placeholder="Type here" />
</KeyboardAvoidingView>

// Dismiss keyboard on tap
<TouchableWithoutFeedback onPress={Keyboard.dismiss}>
  <View style={{ flex: 1 }}>{/* content */}</View>
</TouchableWithoutFeedback>

// Listen to keyboard events
useEffect(() => {
  const showSub = Keyboard.addListener('keyboardDidShow', (e) => {
    console.log('Keyboard height:', e.endCoordinates.height);
  });
  return () => showSub.remove();
}, []);
```

For more control, use `react-native-keyboard-aware-scroll-view` or `react-native-keyboard-controller`.

---

## 34. What is `InteractionManager`?

`InteractionManager` allows you to schedule expensive tasks to run after animations and interactions have completed, keeping the UI responsive.

```jsx
import { InteractionManager } from 'react-native';

useEffect(() => {
  const task = InteractionManager.runAfterInteractions(() => {
    // Expensive operation (data processing, large state update)
    loadHeavyData();
  });

  return () => task.cancel();
}, []);
```

---

## 35. How do you handle app state (foreground/background)?

```jsx
import { AppState } from 'react-native';

useEffect(() => {
  const subscription = AppState.addEventListener('change', (nextState) => {
    if (nextState === 'active') {
      // App came to foreground
      refreshData();
    } else if (nextState === 'background') {
      // App went to background
      saveState();
    }
  });

  return () => subscription.remove();
}, []);
```

App states: `active`, `background`, `inactive` (iOS only, during transitions).

---

## 36. What is the difference between `expo-router` and React Navigation?

| Feature | React Navigation | Expo Router |
|---|---|---|
| Routing model | Configuration-based | File-based (like Next.js) |
| Setup | Manual navigator setup | Automatic from file structure |
| Deep linking | Manual configuration | Automatic from file paths |
| Type safety | Manual typing | Automatic route types |
| Web support | Limited | Built-in |
| Built on | Standalone | React Navigation (under the hood) |

Expo Router uses the file system to define routes:
```
app/
├── index.tsx       → /
├── about.tsx       → /about
├── user/
│   └── [id].tsx    → /user/:id
└── _layout.tsx     → layout wrapper
```

---

## 37. How do you handle accessibility in React Native?

```jsx
<TouchableOpacity
  accessible={true}
  accessibilityLabel="Submit order"
  accessibilityHint="Submits your order and takes you to confirmation"
  accessibilityRole="button"
  accessibilityState={{ disabled: false }}
>
  <Text>Submit</Text>
</TouchableOpacity>

// Dynamic announcements
import { AccessibilityInfo } from 'react-native';
AccessibilityInfo.announceForAccessibility('Order submitted successfully');
```

Key accessibility props:
- `accessible` — marks as an accessibility element
- `accessibilityLabel` — text read by screen readers
- `accessibilityHint` — describes what happens after interaction
- `accessibilityRole` — `button`, `header`, `link`, `image`, etc.
- `accessibilityState` — `disabled`, `selected`, `checked`, `busy`
- `accessibilityValue` — `min`, `max`, `now`, `text` for sliders/progress

---

## 38. What are some common performance issues in React Native?

1. **Large lists without virtualization** — use `FlatList` instead of `ScrollView`
2. **JS thread overload** — heavy computation blocking UI updates
3. **Excessive re-renders** — use `React.memo`, `useMemo`, `useCallback`
4. **Large images** — resize and cache images properly
5. **Too many bridge calls** — batch native calls, use New Architecture
6. **Inline styles/functions** — create them outside render
7. **Console.log in production** — remove or disable logging
8. **Unoptimized animations** — use `useNativeDriver` or Reanimated
9. **Memory leaks** — clean up subscriptions and listeners in effects
10. **Slow navigation transitions** — use native stack navigator, lazy load screens

---

## 39. How do you handle testing in React Native?

| Type | Tool |
|---|---|
| Unit tests | Jest |
| Component tests | React Native Testing Library |
| E2E tests | Detox, Maestro |
| Snapshot tests | Jest snapshots |

```jsx
// Component test with React Native Testing Library
import { render, fireEvent } from '@testing-library/react-native';

test('button press increments counter', () => {
  const { getByText } = render(<Counter />);
  const button = getByText('Increment');
  fireEvent.press(button);
  expect(getByText('Count: 1')).toBeTruthy();
});
```

---

## 40. What is the difference between `StatusBar` component and `expo-status-bar`?

```jsx
// React Native StatusBar
import { StatusBar } from 'react-native';
<StatusBar barStyle="dark-content" backgroundColor="#fff" />

// Expo StatusBar (simpler API)
import { StatusBar } from 'expo-status-bar';
<StatusBar style="dark" />
```

`expo-status-bar` provides a simpler API and automatically handles platform differences. The `style` prop accepts `auto`, `light`, `dark`, or `inverted`.

---

## 41. How do you handle internationalization (i18n)?

Popular libraries:
- **`i18next` + `react-i18next`** — full-featured i18n
- **`expo-localization`** — detect device locale
- **`react-intl`** — internationalization with ICU message syntax

```jsx
import * as Localization from 'expo-localization';
import i18n from 'i18next';
import { useTranslation } from 'react-i18next';

i18n.init({
  lng: Localization.locale,
  resources: {
    en: { translation: { greeting: 'Hello' } },
    es: { translation: { greeting: 'Hola' } },
  },
});

function App() {
  const { t } = useTranslation();
  return <Text>{t('greeting')}</Text>;
}
```

---

## 42. What are Turbo Modules?

Turbo Modules are the New Architecture replacement for legacy Native Modules. Key improvements:

- **Lazy initialization** — modules load only when first used, improving startup time
- **JSI-based** — direct synchronous communication without the bridge
- **Type-safe** — generated from typed specs using Codegen
- **Shared ownership** — native objects can be shared between JS and native without serialization

---

## 43. What is Fabric?

Fabric is the New Architecture rendering system that replaces the legacy UI manager.

Key improvements:
- **Synchronous rendering** — allows measuring layout synchronously
- **Concurrent features** — supports React 18 concurrent rendering
- **Shared C++ core** — cross-platform rendering logic
- **Direct JS-to-native calls** — no bridge serialization for UI operations
- **Better Suspense support** — integrates with React's concurrent features

---

## 44. How do you handle background tasks?

```jsx
// Expo Background Fetch
import * as BackgroundFetch from 'expo-background-fetch';
import * as TaskManager from 'expo-task-manager';

TaskManager.defineTask('BACKGROUND_FETCH', async () => {
  const data = await fetchNewData();
  return data ? BackgroundFetch.BackgroundFetchResult.NewData
              : BackgroundFetch.BackgroundFetchResult.NoData;
});

await BackgroundFetch.registerTaskAsync('BACKGROUND_FETCH', {
  minimumInterval: 15 * 60, // 15 minutes
});
```

Background task capabilities vary by platform. iOS is more restrictive than Android.

---

## 45. What is the difference between `require` and `import` for assets?

```jsx
// require — resolved at build time, bundled into the app
<Image source={require('./assets/logo.png')} />

// URI — loaded at runtime from network or file system
<Image source={{ uri: 'https://example.com/logo.png' }} style={{ width: 100, height: 100 }} />
```

`require` is used for local assets and is processed by the bundler (Metro). It includes the image dimensions automatically. Remote URIs require explicit width and height.

---

## 46. How do you handle app updates?

**Over-the-air (JS only):**
- EAS Update (Expo)
- CodePush (App Center)

**Native updates (store submission required):**
- Use `expo-updates` or `react-native-code-push` to check for updates
- Link to app store for mandatory native updates

```jsx
import * as Updates from 'expo-updates';

const checkForUpdate = async () => {
  const update = await Updates.checkForUpdateAsync();
  if (update.isAvailable) {
    await Updates.fetchUpdateAsync();
    await Updates.reloadAsync();
  }
};
```

---

## 47. What are Config Plugins in Expo?

Config Plugins allow you to customize native project configuration without writing native code directly. They modify `ios/` and `android/` directories during the prebuild step.

```js
// app.config.js
export default {
  plugins: [
    ['expo-camera', { cameraPermission: 'Allow camera access' }],
    ['expo-location', { locationWhenInUsePermission: 'Allow location' }],
    './my-custom-plugin',
  ],
};
```

This enables using native features while staying in the managed Expo workflow.

---

## 48. How do you handle state management in React Native?

State management approaches are the same as React.js:

- **`useState` / `useReducer`** — local component state
- **Context API** — global state (theme, auth, language)
- **Redux Toolkit** — complex global state with middleware
- **Zustand** — lightweight global state
- **Jotai / Recoil** — atomic state management
- **TanStack Query** — server state management
- **MobX** — observable-based state management
- **Legend State** — high-performance reactive state

For React Native specifically, consider persistence:
- Zustand with `persist` middleware + MMKV
- Redux Persist + AsyncStorage or MMKV
- TanStack Query with `persistQueryClient`

---

## 49. What are some common React Native anti-patterns?

1. **Using `ScrollView` for long lists** — use `FlatList` instead
2. **Not using `useNativeDriver`** for animations
3. **Storing sensitive data in AsyncStorage** — use SecureStore or encrypted storage
4. **Not handling keyboard properly** — use `KeyboardAvoidingView`
5. **Ignoring platform differences** — test on both iOS and Android
6. **Not cleaning up listeners** — causes memory leaks
7. **Blocking the JS thread** — offload heavy work to native or use `InteractionManager`
8. **Hardcoding dimensions** — use responsive values with `useWindowDimensions` or percentage-based layouts
9. **Not handling loading/error states** — always show feedback for async operations
10. **Ignoring accessibility** — use accessibility props for screen reader support

---

## 50. What questions should you ask about a company's React Native setup?

When interviewing, consider asking:
- Are you using Expo or bare React Native?
- Which navigation library do you use?
- Have you migrated to the New Architecture?
- What state management solution do you use?
- How do you handle OTA updates?
- What is your testing strategy (unit, integration, E2E)?
- How do you handle CI/CD and builds?
- Do you share code between platforms (web, iOS, Android)?
- What is your minimum supported OS version?
- How do you handle native module development?
