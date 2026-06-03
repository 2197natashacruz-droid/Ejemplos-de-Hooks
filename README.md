# ⚛️ React Hooks — Los 10 más usados

Guía de referencia rápida con ejemplos prácticos listos para copiar.

---

## Tabla de contenidos

| # | Hook | Para qué sirve |
|---|------|----------------|
| 1 | [`useState`](#1-usestate) | Estado local del componente |
| 2 | [`useEffect`](#2-useeffect) | Efectos secundarios y ciclo de vida |
| 3 | [`useRef`](#3-useref) | Referencia al DOM o valor persistente |
| 4 | [`useContext`](#4-usecontext) | Consumir un contexto global |
| 5 | [`useReducer`](#5-usereducer) | Estado complejo con lógica centralizada |
| 6 | [`useMemo`](#6-usememo) | Memoizar valores calculados |
| 7 | [`useCallback`](#7-usecallback) | Memoizar funciones |
| 8 | [`useLayoutEffect`](#8-uselayouteffect) | Efecto síncrono antes del pintado |
| 9 | [`useId`](#9-useid) | Generar IDs únicos accesibles |
| 10 | [`useTransition`](#10-usetransition) | Marcar actualizaciones como no urgentes |

---

## 1. `useState`

Permite agregar **estado local** a un componente funcional. Devuelve el valor actual y una función para actualizarlo.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0); // valor inicial: 0

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicks: {count}
    </button>
  );
}
```

> **Cuándo usarlo:** siempre que un componente necesite recordar algo entre renders (un input, un toggle, un contador, etc.).

---

## 2. `useEffect`

Ejecuta **efectos secundarios** (fetch, suscripciones, timers) después del render. El array de dependencias controla cuándo se ejecuta.

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    fetch(`/api/users/${userId}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(() => {}); // ignora AbortError

    return () => controller.abort(); // limpieza al desmontar
  }, [userId]); // se re-ejecuta si userId cambia

  return <div>{user?.name}</div>;
}
```

> **Regla de dependencias:** `[]` = solo al montar · `[x]` = cuando cambia `x` · sin array = en cada render.

---

## 3. `useRef`

Devuelve un objeto `{ current }` que **persiste entre renders sin causar re-render**. Útil para referenciar nodos del DOM o guardar valores mutables.

```jsx
import { useRef } from 'react';

function VideoPlayer() {
  const videoRef = useRef(null);

  const handlePlay = () => {
    videoRef.current.play(); // acceso directo al DOM
  };

  return (
    <>
      <video ref={videoRef} src="/video.mp4" />
      <button onClick={handlePlay}>▶ Play</button>
    </>
  );
}
```

> **Cuándo usarlo:** foco en inputs, scroll programático, timers, guardar el valor previo de un state.

---

## 4. `useContext`

Consume un **contexto de React** sin necesidad de prop drilling. Ideal para datos globales como tema, idioma o usuario autenticado.

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Crear el contexto
const ThemeContext = createContext('light');

// 2. Proveer el contexto
function App() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={theme}>
      <Page />
    </ThemeContext.Provider>
  );
}

// 3. Consumir desde cualquier hijo
function Page() {
  const theme = useContext(ThemeContext);
  return <div className={theme}>Tema actual: {theme}</div>;
}
```

> **Cuándo usarlo:** autenticación, tema visual, idioma (i18n), configuración global.

---

## 5. `useReducer`

Alternativa a `useState` para **lógica de estado compleja**. Centraliza las actualizaciones en una función `reducer` pura.

```jsx
import { useReducer } from 'react';

const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment': return { ...state, count: state.count + state.step };
    case 'decrement': return { ...state, count: state.count - state.step };
    case 'setStep':   return { ...state, step: action.payload };
    case 'reset':     return initialState;
    default:          return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <>
      <p>Contador: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </>
  );
}
```

> **Cuándo usarlo:** cuando el estado tiene múltiples sub-valores relacionados o las transiciones son complejas.

---

## 6. `useMemo`

**Memoriza el resultado** de una función costosa y solo la recalcula cuando cambian sus dependencias.

```jsx
import { useState, useMemo } from 'react';

function ProductList({ products, filter }) {
  // solo se recalcula si products o filter cambian
  const filtered = useMemo(() => {
    return products.filter(p =>
      p.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [products, filter]);

  return (
    <ul>
      {filtered.map(p => <li key={p.id}>{p.name}</li>)}
    </ul>
  );
}
```

> **Cuándo usarlo:** cálculos pesados, filtros sobre listas grandes, valores derivados que se usan en renders frecuentes.

---

## 7. `useCallback`

Devuelve una **función memoizada** que solo cambia si sus dependencias cambian. Evita que los hijos se re-rendericen innecesariamente.

```jsx
import { useState, useCallback } from 'react';

function Parent() {
  const [count, setCount] = useState(0);

  // sin useCallback: nueva función en cada render → Child siempre re-renderiza
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []); // sin deps: misma referencia siempre

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>+{count}</button>
      <Child onClick={handleClick} />
    </>
  );
}

const Child = React.memo(({ onClick }) => {
  console.log('Child renderizó');
  return <button onClick={onClick}>Hijo</button>;
});
```

> **Cuándo usarlo:** junto a `React.memo` en componentes hijos, o cuando una función se pasa como dependencia de otro hook.

---

## 8. `useLayoutEffect`

Igual que `useEffect`, pero se ejecuta **de forma síncrona después de las mutaciones del DOM y antes del pintado**. Útil para medir elementos o aplicar estilos sin parpadeo.

```jsx
import { useRef, useLayoutEffect, useState } from 'react';

function Tooltip({ text }) {
  const ref = useRef(null);
  const [width, setWidth] = useState(0);

  useLayoutEffect(() => {
    // el DOM ya mutó pero el navegador aún no pintó
    setWidth(ref.current.getBoundingClientRect().width);
  }, [text]);

  return (
    <span ref={ref}>
      {text} <em style={{ fontSize: 11 }}>({width}px)</em>
    </span>
  );
}
```

> **Cuándo usarlo:** medir el DOM, posicionar tooltips/popovers, evitar el flash de contenido desplazado. En la mayoría de casos, prefiere `useEffect`.

---

## 9. `useId`

Genera un **ID único y estable** que funciona tanto en servidor como cliente (SSR). Ideal para conectar labels con inputs de forma accesible.

```jsx
import { useId } from 'react';

function FormField({ label, type = 'text' }) {
  const id = useId(); // ej: ":r3:"

  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input id={id} type={type} />
    </div>
  );
}

// Se puede usar múltiples veces en el mismo componente
function SignupForm() {
  const baseId = useId();
  return (
    <>
      <label htmlFor={`${baseId}-email`}>Email</label>
      <input id={`${baseId}-email`} type="email" />

      <label htmlFor={`${baseId}-pass`}>Contraseña</label>
      <input id={`${baseId}-pass`} type="password" />
    </>
  );
}
```

> **Cuándo usarlo:** formularios accesibles, aria-labelledby, cualquier atributo que requiera IDs únicos en el DOM.

---

## 10. `useTransition`

Marca una actualización de estado como **no urgente**, permitiendo que React priorice interacciones más importantes (como el input del usuario) sobre renders lentos.

```jsx
import { useState, useTransition } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    setQuery(e.target.value); // urgente: actualiza el input al instante

    startTransition(() => {
      // no urgente: React puede interrumpir esto si el usuario sigue escribiendo
      setResults(expensiveSearch(e.target.value));
    });
  };

  return (
    <>
      <input value={query} onChange={handleChange} placeholder="Buscar..." />
      {isPending && <span>Buscando...</span>}
      <ResultList results={results} />
    </>
  );
}
```

> **Cuándo usarlo:** filtros en tiempo real, navegación entre tabs, cualquier actualización costosa que no debe bloquear la UI.

---

## Referencia rápida

| Hook | Re-render al cambiar | Persiste entre renders | Caso de uso principal |
|------|---------------------|------------------------|----------------------|
| `useState` | ✅ | ✅ | Estado local |
| `useReducer` | ✅ | ✅ | Estado complejo |
| `useRef` | ❌ | ✅ | DOM / valor mutable |
| `useEffect` | ❌ | — | Efectos, fetch, subs |
| `useLayoutEffect` | ❌ | — | Medir DOM pre-paint |
| `useContext` | ✅ | — | Estado global |
| `useMemo` | ❌ | — | Valores memoizados |
| `useCallback` | ❌ | — | Funciones memoizadas |
| `useId` | ❌ | ✅ | IDs únicos accesibles |
| `useTransition` | ✅ | — | Renders no urgentes |

---

> Creado con ❤️ para el proyecto **Chinchintirapie** · React 18+
