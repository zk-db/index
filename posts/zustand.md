# Zustand

nextjs client global state manage：

- context
- zustand


## context
1. create context
```tsx
// store.ts
import { create } from "zustand";

interface AuthState {
  user: string | null;
  setUser: (user: string | null) => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));
```
2. use
```tsx
// app/page.tsx
"use client"
import { useAuth } from "../AuthContext";

export default function Page() {
  const { user, setUser } = useAuth();

  return (
    <div>
      <p>Current user: {user ?? "guest"}</p>
      <button onClick={() => setUser("Zeki")}>Login as Zeki</button>
    </div>
  );
}
```

## Zustand
1. create
```tsx
// store.ts
import { create } from "zustand";

interface AuthState {
  user: string | null;
  setUser: (user: string | null) => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));
```
2. use
```tsx
// app/page.tsx
"use client";
import { useAuthStore } from "../store";

export default function Page() {
  const { user, setUser } = useAuthStore();

  return (
    <div>
      <p>Current user: {user ?? "guest"}</p>
      <button onClick={() => setUser("Zeki")}>Login as Zeki</button>
    </div>
  );
}
``

3. persistent
```tsx
// store.ts
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface AuthState {
  user: string | null;
  setUser: (user: string | null) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      setUser: (user) => set({ user }),
    }),
    {
      name: "auth-storage", // 存到 localStorage 的 key
    }
  )
);
``


