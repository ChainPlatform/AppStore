# 🧠 AppStore

A lightweight, dependency-free global store with **persistence**, **hydration**, and **subscription** support — designed for React Native and Web environments.  
It lets you manage multiple namespaces (like `theme`, `user`, `settings`, etc) and persist them easily using your own storage backend.

<p align="center">
  <a href="https://github.com/ChainPlatform/AppStore/blob/HEAD/LICENSE">
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" />
  </a>
  <a href="https://www.npmjs.com/package/@chainplatform/appstore">
    <img src="https://img.shields.io/npm/v/@chainplatform/appstore?color=brightgreen&label=npm%20package" />
  </a>
  <a href="https://www.npmjs.com/package/@chainplatform/appstore">
    <img src="https://img.shields.io/npm/dt/@chainplatform/appstore.svg" />
  </a>
  <a href="https://www.npmjs.com/package/@chainplatform/appstore">
    <img src="https://img.shields.io/badge/platform-android%20%7C%20ios%20%7C%20web-blue" />
  </a>
  <a href="https://github.com/ChainPlatform/AppStore/pulls">
    <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs welcome!" />
  </a>
  <a href="https://x.com/intent/follow?screen_name=doansan">
    <img src="https://img.shields.io/twitter/follow/doansan.svg?label=Follow%20@doansan" alt="Follow @doansan"></img>
  </a>
</p>

---

## ✨ Features

- ✅ Multi-namespace store management (`AppStore.use("theme")`, `AppStore.use("user")`)
- 💾 Persistent data via pluggable storage adapter
- 🔁 Auto hydration (load from storage)
- 👂 Reactive subscriptions with `subscribe()` and `onHydrated()`
- ⚙️ Custom logging and debounced persistence
- 🔒 Works with encrypted or plain storage

---

## 🚀 Installation

```bash
npm install @chainplatform/appstore
```

or

```bash
yarn add @chainplatform/appstore
```

---

## ⚙️ Quick Setup

You must configure your storage adapter once before using any stores.

### Example with Chain SDK

```js
import AppStore from "@chainplatform/appstore";
import { saveStorage, retrieveStorage, removeStorage } from "@chainplatform/sdk";

const customStorage = {
  get: async (key) => await retrieveStorage(key, { encrypted: true }),
  set: async (key, val) => await saveStorage(key, val, { encrypted: true }),
  remove: async (key) => await removeStorage(key, { encrypted: true }),
};

// Configure once at app startup
AppStore.configure({
  storage: customStorage,
  log: true, // or false, or a custom logger function
});
```

---

## 🧩 Usage Example

```js
// Create or access a store
const themeStore = AppStore.use("theme", { encrypted: true });

// Hydrate data from persistent storage
await themeStore.hydrate();

// Subscribe to changes
const unsubscribe = themeStore.subscribe((newVal, oldVal) => {
  console.log("Theme changed:", newVal);
});

// Set data
themeStore.set({ mode: "dark", primary: "#3498db" });

// Clear only in-memory data
themeStore.clear();

// Clear both in-memory and storage
await themeStore.clearStorage();

// Unsubscribe when needed
unsubscribe();
```

---

## 🔄 Manage Multiple Stores

```js
const userStore = AppStore.use("user");
const drawerStore = AppStore.use("drawer");

// Load all stores in parallel
await AppStore.hydrateAll();

// Clear all memory data (not storage)
AppStore.clearAll();
```

---

## 🧱 API Reference

### `AppStore.configure(options)`
Configure global settings for all stores.  
Should be called **once**, usually at app startup.

| Option     | Type                | Description |
|-------------|---------------------|-------------|
| `storage`   | `{ get, set, remove }` | Custom async storage adapter |
| `log`       | `boolean \| function` | Enables or overrides logging (default: `false`) |

---

### `AppStore.use(namespace, options)`
Create or get a store by namespace.

| Param | Type | Description |
|--------|------|-------------|
| `namespace` | `string` | Unique key for store (e.g. `"theme"`, `"user"`) |

Returns a `SingleStore` instance.

---

### `SingleStore` API

| Method | Description |
|--------|-------------|
| `hydrate()` | Load stored data and mark as hydrated |
| `subscribe(cb, { fireImmediately? })` | Listen for data changes |
| `onHydrated(cb)` | Called once when hydration completes |
| `set(data)` | Set data |
| `clear()` | Clear in-memory data |
| `clearStorage()` | Clear both in-memory and persistent storage |
| `value` | Current store value |
| `initialized` | Whether the store has been hydrated |

---

## 🪵 Logging

You can pass a logger function or enable default console logging:

```js
AppStore.configure({
  storage: customStorage,
  log: (msg, ...args) => console.debug("[AppStore]", msg, ...args),
});

// or simply:
AppStore.configure({ storage: customStorage, log: true });
```

All log messages include namespace tags like `[theme] hydrated`.

---

## 🧠 Tips

- Call `AppStore.configure()` **only once globally** — all stores share the same storage and logger.
- Each store persists independently using its own `namespace` key.
- To reset everything quickly:  
  ```js
  AppStore.clearAll();
  await customStorage.remove("theme");
  await customStorage.remove("user");
  ```

---

## 🧪 Unit Testing Example (Jest)

```js
import AppStore from "../src/helpers/AppStore";

describe("AppStore", () => {
  const mockStorage = {
    get: jest.fn(async () => JSON.stringify({ theme: "dark" })),
    set: jest.fn(async () => {}),
    remove: jest.fn(async () => {}),
  };

  beforeAll(() => {
    AppStore.configure({ storage: mockStorage, log: false });
  });

  it("hydrates and sets data", async () => {
    const store = AppStore.use("theme");
    await store.hydrate();
    expect(store.value).toEqual({ theme: "dark" });

    store.set({ theme: "light" });
    expect(store.value.theme).toBe("light");
  });
});
```

---

## 📜 License

MIT © Chain Platform
