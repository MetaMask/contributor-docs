# Frontend Performance Optimization Guidelines

Guidelines for optimizing React and Redux performance.

> **🔥 Performance Audit Tracking:** This document is actively maintained alongside the [Frontend Performance Audit Epic (#6571)](https://github.com/MetaMask/MetaMask-planning/issues/6571). See linked tickets for current remediation status.

## Executive Summary: Priority Action Items

Performance anti-patterns have propagated across the MetaMask Extension codebase, causing severe slowdowns under heavy load. The following table summarizes the **highest-impact issues** that should be addressed first:

### 🔴 Critical Priority (P0) — Fix Immediately

| Anti-Pattern | Count | Impact | Tracking |
|--------------|-------|--------|----------|
| `useSelector(selector, isEqual)` | 59 | Deep comparison on EVERY render = 5,900+ comparisons/sec | [#6536](https://github.com/MetaMask/MetaMask-planning/issues/6536) |
| `JSON.stringify` in useEffect deps | 15+ | Expensive serialization on EVERY render | [#6545](https://github.com/MetaMask/MetaMask-planning/issues/6545) |
| Unnecessary `createDeepEqualSelector` | 141 | `isEqual` runs on every selector evaluation | [#6537](https://github.com/MetaMask/MetaMask-planning/issues/6537) |

### 🟠 High Priority (P1) — Fix in Current Sprint

| Anti-Pattern | Count | Impact | Tracking |
|--------------|-------|--------|----------|
| `key={index}` on dynamic lists | 72 | Reconciliation bugs, state corruption | [#6523](https://github.com/MetaMask/MetaMask-planning/issues/6523) |
| `Object.values().find()` | 52+ | O(n) search on every call | [#6539](https://github.com/MetaMask/MetaMask-planning/issues/6539) |
| Multiple `useSelector` calls | 200+ | Redundant subscriptions | [#6524](https://github.com/MetaMask/MetaMask-planning/issues/6524) |
| Routes not lazy-loaded | 15+ | Large initial bundle, slow LCP | [#6547](https://github.com/MetaMask/MetaMask-planning/issues/6547) |

### Expected Core Web Vitals Impact

| Metric | Current | Target | Improvement |
|--------|---------|--------|-------------|
| **INP** (Token Search) | 200-400ms | 30-60ms | **85%** |
| **INP** (Swap Quotes) | 300-600ms | 100-200ms | **65%** |
| **LCP** (Popup Open) | 800-1200ms | &lt;500ms | **50%** |
| **CLS** (List Scroll) | 0.1-0.3 | &lt;0.05 | **80%** |

---

## How Anti-Patterns Propagate

Understanding propagation prevents future issues:

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Initial Pattern: Developer adds useSelector(..., isEqual)   │
│     to fix a re-render bug without fixing the root cause        │
│                              ↓                                   │
│  2. Code Copying: Pattern spreads as reference implementation   │
│     Swaps team copies → Bridge team copies → 59 occurrences     │
│                              ↓                                   │
│  3. No Guardrails: ESLint doesn't flag, reviews don't catch     │
│     Each addition seems harmless in isolation                   │
│                              ↓                                   │
│  4. Compounding Impact: 59 deep comparisons × 100 updates/sec   │
│     = 5,900 deep traversals per second under load               │
│                              ↓                                   │
│  5. Power User Pain: Users with many accounts/tokens hit walls  │
│     Performance degrades non-linearly with data volume          │
└─────────────────────────────────────────────────────────────────┘
```

**Prevention:**
- ESLint rule to flag `useSelector(..., isEqual)` ([#6565](https://github.com/MetaMask/MetaMask-planning/issues/6565))
- Performance baselines in CI ([#6566](https://github.com/MetaMask/MetaMask-planning/issues/6566))
- Code review guidelines ([#6567](https://github.com/MetaMask/MetaMask-planning/issues/6567))

---

## Table of Contents

### Chapter 1: Rendering Performance

- [Optimizing Lists and Large Datasets](#optimizing-lists-and-large-datasets)
  - [Use Proper Keys](#use-proper-keys)
  - [❌ Anti-Pattern: Using Index as Key for Dynamic Lists](#-anti-pattern-using-index-as-key-for-dynamic-lists)
  - [Virtualize Long Lists](#virtualize-long-lists)
  - [Pagination and Infinite Scroll](#pagination-and-infinite-scroll)
- [Code Splitting and Lazy Loading](#code-splitting-and-lazy-loading)
  - [Use React.lazy for Route-Based Splitting](#use-reactlazy-for-route-based-splitting)
  - [Lazy Load Heavy Components](#lazy-load-heavy-components)
- [Memoization and Offloading of Computations](#memoization-and-offloading-of-computations)
  - [❌ Anti-Pattern: Creating Maps/Objects During Render](#-anti-pattern-creating-mapsobjects-during-render)
  - [❌ Anti-Pattern: Expensive Operations Without Memoization](#-anti-pattern-expensive-operations-without-memoization)
  - [Strategy: Move Complex Computations to Selectors](#strategy-move-complex-computations-to-selectors)
  - [Strategy: Use Web Workers for Heavy Computations](#strategy-use-web-workers-for-heavy-computations)
  - [Strategy: Debounce Frequent Updates](#strategy-debounce-frequent-updates)

### Chapter 2: Hooks &amp; Effects

- [Hook Optimizations](#hook-optimizations)
  - [Don't Overuse useEffect](#dont-overuse-useeffect)
  - [Minimize useEffect Dependencies](#minimize-useeffect-dependencies)
  - [🔴 CRITICAL: JSON.stringify in useEffect Dependencies](#-critical-jsonstringify-in-useeffect-dependencies)
  - [❌ Anti-Pattern: useEffect with Incomplete Dependencies (Stale Closures)](#-anti-pattern-useeffect-with-incomplete-dependencies-stale-closures)
  - [❌ Anti-Pattern: Wrong Dependencies in useMemo/useCallback](#-anti-pattern-wrong-dependencies-in-usememousecallback)
  - [❌ Anti-Pattern: Cascading useEffect Chains](#-anti-pattern-cascading-useeffect-chains)
  - [❌ Anti-Pattern: Conditional Early Return with All Dependencies](#-anti-pattern-conditional-early-return-with-all-dependencies)
  - [❌ Anti-Pattern: Regular Variables Instead of useRef](#-anti-pattern-regular-variables-instead-of-useref)
- [Non-Deterministic Hook Execution](#non-deterministic-hook-execution)
  - [❌ Anti-Pattern: Conditional Hook Calls](#-anti-pattern-conditional-hook-calls)
  - [❌ Anti-Pattern: Dynamic Hook Creation](#-anti-pattern-dynamic-hook-creation)
- [Cascading Re-renders from Hook Dependencies](#cascading-re-renders-from-hook-dependencies)
  - [❌ Anti-Pattern: Chain Reaction Re-renders](#-anti-pattern-chain-reaction-re-renders)
  - [Strategy: Isolate Hook Dependencies](#strategy-isolate-hook-dependencies)
  - [Strategy: Use Component Composition to Prevent Re-renders](#use-component-composition-to-prevent-re-renders)
- [Async Operations and Cleanup](#async-operations-and-cleanup)
  - [❌ Anti-Pattern: State Updates After Component Unmount](#-anti-pattern-state-updates-after-component-unmount)
  - [❌ Anti-Pattern: Missing AbortController for Fetch Requests](#-anti-pattern-missing-abortcontroller-for-fetch-requests)
  - [❌ Anti-Pattern: Missing Cleanup for Intervals/Subscriptions](#-anti-pattern-missing-cleanup-for-intervalssubscriptions)
  - [❌ Anti-Pattern: Large Object Retention in Closures](#-anti-pattern-large-object-retention-in-closures)

### Chapter 3: State Management

- [🔴 CRITICAL: Never Use useSelector with isEqual Comparator](#-critical-never-use-useselector-with-isequal-comparator)
- [Advanced Selector Patterns](#advanced-selector-patterns)
  - [❌ Anti-Pattern: Identity Functions as Output Selectors](#-anti-pattern-identity-functions-as-output-selectors)
  - [❌ Anti-Pattern: Selectors That Return Entire State Objects](#-anti-pattern-selectors-that-return-entire-state-objects)
  - [❌ Anti-Pattern: Selectors Without Proper Input Selectors](#-anti-pattern-selectors-without-proper-input-selectors)
  - [Best Practices for Reselect Selectors](#best-practices-for-reselect-selectors)
  - [Use `createDeepEqualSelector` sparingly](#use-createdeepequalselector-sparingly)
  - [Combine related selectors into one memoized selector](#combine-related-selectors-into-one-memoized-selector)
  - [❌ Anti-Pattern: Inline Selector Functions in useSelector](#-anti-pattern-inline-selector-functions-in-useselector)
  - [❌ Anti-Pattern: Multiple useSelector Calls Selecting Same State Slice](#-anti-pattern-multiple-useselector-calls-selecting-same-state-slice)
  - [❌ Anti-Pattern: Inefficient Use of `Object.values()` and `Object.keys()` in Selectors](#-anti-pattern-inefficient-use-of-objectvalues-and-objectkeys-in-selectors)
  - [❌ Anti-Pattern: Deep Property Access in Selectors](#-anti-pattern-deep-property-access-in-selectors)
  - [❌ Anti-Pattern: Repeated Object Traversal in Selectors](#-anti-pattern-repeated-object-traversal-in-selectors)
  - [❌ Anti-Pattern: Selectors That Reorganize Nested State](#-anti-pattern-selectors-that-reorganize-nested-state)
  - [❌ Anti-Pattern: Filtering/Searching Through Nested Objects](#-anti-pattern-filteringsearching-through-nested-objects)

### Chapter 4: React Compiler

- [React Compiler Considerations](#react-compiler-considerations)
  - [What React Compiler Does](#what-react-compiler-does)
  - [React Compiler Assumptions](#react-compiler-assumptions)
  - [React Compiler Limitations](#react-compiler-limitations)
  - [When Manual Memoization is Still Required](#when-manual-memoization-is-still-required)
  - [Decision Tree: Do You Need Manual Memoization?](#decision-tree-do-you-need-manual-memoization)
  - [Summary: React Compiler Capabilities and Limitations](#summary-react-compiler-capabilities-and-limitations)

---

## Optimizing Lists and Large Datasets

> **Tracking:** [Epic #6523: Fix Array Index Keys](https://github.com/MetaMask/MetaMask-planning/issues/6523)

### Use Proper Keys

```typescript
❌ WRONG: Using array index as key
const TokenList = ({ tokens }: TokenListProps) => {
  return (
    &lt;div&gt;
      {tokens.map((token, index) => (
        &lt;TokenItem key={index} token={token} /&gt; // Bad if list can reorder!
      ))}
    &lt;/div&gt;
  );
};

✅ CORRECT: Use unique, stable identifiers
const TokenList = ({ tokens }: TokenListProps) => {
  return (
    &lt;div&gt;
      {tokens.map(token => (
        &lt;TokenItem key={token.address} token={token} /&gt;
      ))}
    &lt;/div&gt;
  );
};
```

**Key rules:**

- ✅ Use unique IDs from data (address, id, uuid)
- ✅ Keys must be stable across re-renders
- ⚠️ Only use index if list never reorders and items don't have IDs
- ❌ Never use random values or `Math.random()`

### ❌ Anti-Pattern: Using Index as Key for Dynamic Lists

Using array index as key breaks React's reconciliation when lists can be reordered, filtered, or items added/removed.

```typescript
❌ WRONG: Index keys break reconciliation
const TokenList = ({ tokens }: TokenListProps) => {
  // If tokens can be reordered/filtered, this breaks React's reconciliation
  return (
    &lt;div&gt;
      {tokens.map((token, index) => (
        &lt;TokenItem key={index} token={token} /&gt; // Bad!
      ))}
    &lt;/div&gt;
  );
};

✅ CORRECT: Use unique, stable identifiers
const TokenList = ({ tokens }: TokenListProps) => {
  return (
    &lt;div&gt;
      {tokens.map(token => (
        &lt;TokenItem key={token.address} token={token} /&gt; // Good!
      ))}
    &lt;/div&gt;
  );
};
```

**Problems with index keys:**

- React can't track items when list reorders
- State gets mixed up between items
- Causes bugs with form inputs, focus, animations
- Performance issues from unnecessary re-renders

**Solution:** Always use unique, stable identifiers from your data (address, id, uuid). Only use index if the list is static and never reorders.

### Virtualize Long Lists

> **Tracking:** [Epic #6526: List Virtualization](https://github.com/MetaMask/MetaMask-planning/issues/6526)

For lists with 100+ items, use virtualization to only render visible items.

```typescript
❌ WRONG: Rendering 1000+ items at once
const TransactionList = ({ transactions }: TransactionListProps) => {
  return (
    &lt;div className="transaction-list"&gt;
      {transactions.map(tx => (
        &lt;TransactionItem key={tx.hash} transaction={tx} /&gt;
      ))}
    &lt;/div&gt;
  );
};

✅ CORRECT: Use virtualization library
import { FixedSizeList } from 'react-window';

const TransactionList = ({ transactions }: TransactionListProps) => {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    &lt;div style={style}&gt;
      &lt;TransactionItem transaction={transactions[index]} /&gt;
    &lt;/div&gt;
  );

  return (
    &lt;FixedSizeList
      height={600}
      itemCount={transactions.length}
      itemSize={80}
      width="100%"
    &gt;
      {Row}
    &lt;/FixedSizeList&gt;
  );
};
```

```typescript
✅ GOOD: Virtual scrolling for 1000+ assets
import { FixedSizeList } from 'react-window';

const AssetList = ({ assets }: AssetListProps) => {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    &lt;div style={style}&gt;
      &lt;Asset asset={assets[index]} /&gt;
    &lt;/div&gt;
  );

  return (
    &lt;FixedSizeList
      height={600}
      itemCount={assets.length}
      itemSize={80}
      width="100%"
    &gt;
      {Row}
    &lt;/FixedSizeList&gt;
  );
};
```

**Recommended libraries:**

- `react-window` - Lightweight, recommended for most use cases
- `react-virtualized` - More features, larger bundle size

### Pagination and Infinite Scroll

For very large datasets, load data in chunks.

```typescript
✅ GOOD: Paginated data loading
const TransactionList = () => {
  const [page, setPage] = useState(1);
  const { transactions, hasMore, isLoading } = useTransactionsPaginated(page);

  const loadMore = useCallback(() => {
    if (!isLoading &amp;&amp; hasMore) {
      setPage(p => p + 1);
    }
  }, [isLoading, hasMore]);

  return (
    &lt;div&gt;
      {transactions.map(tx => (
        &lt;TransactionItem key={tx.hash} transaction={tx} /&gt;
      ))}
      {hasMore &amp;&amp; (
        &lt;button onClick={loadMore} disabled={isLoading}&gt;
          Load More
        &lt;/button&gt;
      )}
    &lt;/div&gt;
  );
};
```

```typescript
❌ WRONG: Load all assets at once
const AssetList = ({ accountId }: AssetListProps) => {
  const [assets, setAssets] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Loads ALL assets at once - blocks UI
    fetchAllAssets(accountId).then(allAssets => {
      setAssets(allAssets); // 1000+ assets loaded at once!
      setLoading(false);
    });
  }, [accountId]);

  return loading ? &lt;Spinner /&gt; : &lt;div&gt;{assets.map(a => &lt;Asset key={a.id} asset={a} /&gt;}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: Progressive pagination
const AssetList = ({ accountId }: AssetListProps) => {
  const [assets, setAssets] = useState([]);
  const [hasMore, setHasMore] = useState(true);
  const [loading, setLoading] = useState(false);
  const pageRef = useRef(0);
  const PAGE_SIZE = 50;

  const loadPage = useCallback(async () => {
    if (loading || !hasMore) return;

    setLoading(true);
    try {
      const result = await fetchAssetsPage(accountId, pageRef.current, PAGE_SIZE);
      setAssets(prev => [...prev, ...result.items]);
      setHasMore(result.hasMore);
      pageRef.current += 1;
    } finally {
      setLoading(false);
    }
  }, [accountId, loading, hasMore]);

  useEffect(() => {
    // Load first page on mount
    loadPage();
  }, [accountId]); // Reset on account change

  return (
    &lt;div&gt;
      {assets.map(a => &lt;Asset key={a.id} asset={a} /&gt;)}
      {hasMore &amp;&amp; (
        &lt;button onClick={loadPage} disabled={loading}&gt;
          {loading ? 'Loading...' : 'Load More'}
        &lt;/button&gt;
      )}
    &lt;/div&gt;
  );
};
```

---

## Code Splitting and Lazy Loading

> **Tracking:** [Epic #6527: Code Splitting](https://github.com/MetaMask/MetaMask-planning/issues/6527)

### Use React.lazy for Route-Based Splitting

```typescript
❌ WRONG: Import all pages upfront
import Settings from './pages/Settings';
import Tokens from './pages/Tokens';
import Activity from './pages/Activity';

const App = () => {
  return (
    &lt;Routes&gt;
      &lt;Route path="/settings" element={&lt;Settings /&gt;} /&gt;
      &lt;Route path="/tokens" element={&lt;Tokens /&gt;} /&gt;
      &lt;Route path="/activity" element={&lt;Activity /&gt;} /&gt;
    &lt;/Routes&gt;
  );
};

✅ CORRECT: Lazy load pages
import { lazy, Suspense } from 'react';

const Settings = lazy(() => import('./pages/Settings'));
const Tokens = lazy(() => import('./pages/Tokens'));
const Activity = lazy(() => import('./pages/Activity'));

const App = () => {
  return (
    &lt;Suspense fallback={&lt;LoadingSpinner /&gt;}&gt;
      &lt;Routes&gt;
        &lt;Route path="/settings" element={&lt;Settings /&gt;} /&gt;
        &lt;Route path="/tokens" element={&lt;Tokens /&gt;} /&gt;
        &lt;Route path="/activity" element={&lt;Activity /&gt;} /&gt;
      &lt;/Routes&gt;
    &lt;/Suspense&gt;
  );
};
```

### Lazy Load Heavy Components

```typescript
✅ GOOD: Lazy load modals and heavy components
const QRCodeScanner = lazy(() => import('./components/QRCodeScanner'));

const SendToken = () => {
  const [showScanner, setShowScanner] = useState(false);

  return (
    &lt;div&gt;
      &lt;input placeholder="Recipient address" /&gt;
      &lt;button onClick={() => setShowScanner(true)}&gt;
        Scan QR Code
      &lt;/button&gt;

      {showScanner &amp;&amp; (
        &lt;Suspense fallback={&lt;div&gt;Loading scanner...&lt;/div&gt;}&gt;
          &lt;QRCodeScanner onScan={handleScan} /&gt;
        &lt;/Suspense&gt;
      )}
    &lt;/div&gt;
  );
};
```

```typescript
✅ GOOD: Lazy load asset images
const AssetCard = ({ asset }: AssetCardProps) => {
  const [imageLoaded, setImageLoaded] = useState(false);
  const imgRef = useRef&lt;HTMLImageElement&gt;(null);

  useEffect(() => {
    if (!imgRef.current) return;

    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setImageLoaded(true);
          observer.disconnect();
        }
      },
      { rootMargin: '50px' } // Start loading 50px before visible
    );

    observer.observe(imgRef.current);

    return () => observer.disconnect();
  }, []);

  return (
    &lt;div&gt;
      &lt;div&gt;{asset.name}&lt;/div&gt;
      {imageLoaded ? (
        &lt;img src={asset.imageUrl} alt={asset.name} /&gt;
      ) : (
        &lt;div className="placeholder"&gt;Loading image...&lt;/div&gt;
      )}
    &lt;/div&gt;
  );
};
```

## Memoization and Offloading of Computations

### ❌ Anti-Pattern: Creating Maps/Objects During Render

Creating Maps, Sets, or complex objects during render blocks the main thread.

```typescript
❌ WRONG: Creating Map and mapping arrays on every render
const UnconnectedAccountAlert = () => {
  const internalAccounts = useSelector(getInternalAccounts);
  const connectedAccounts = useSelector(getOrderedConnectedAccountsForActiveTab);

  // Map creation runs on every render
  const internalAccountsMap = new Map(
    internalAccounts.map((acc) => [acc.address, acc]),
  );

  // Array mapping runs on every render
  const connectedAccountsWithName = connectedAccounts.map((account) => ({
    ...account,
    name: internalAccountsMap.get(account.address)?.metadata.name,
  }));

  return &lt;div&gt;{connectedAccountsWithName.map(...)}&lt;/div&gt;;
};
```

**Problems:**

- Map creation runs on every render
- Array mapping runs on every render
- Object spreading creates new objects
- Expensive operations blocking render

```typescript
✅ CORRECT: Memoize expensive computations
const UnconnectedAccountAlert = () => {
  const internalAccounts = useSelector(getInternalAccounts);
  const connectedAccounts = useSelector(getOrderedConnectedAccountsForActiveTab);

  // Memoize Map creation
  const internalAccountsMap = useMemo(
    () => new Map(internalAccounts.map((acc) => [acc.address, acc])),
    [internalAccounts]
  );

  // Memoize array transformation
  const connectedAccountsWithName = useMemo(
    () =>
      connectedAccounts.map((account) => ({
        ...account,
        name: internalAccountsMap.get(account.address)?.metadata.name,
      })),
    [connectedAccounts, internalAccountsMap]
  );

  return &lt;div&gt;{connectedAccountsWithName.map(...)}&lt;/div&gt;;
};
```

### ❌ Anti-Pattern: Expensive Operations Without Memoization

Complex map/filter/reduce operations during render block the main thread.

```typescript
❌ WRONG: Expensive computation on every render
const AssetDashboard = ({ assets, filters }: AssetDashboardProps) => {
  // These run on EVERY render, even if assets/filters haven't changed
  const filtered = assets
    .filter(asset => matchesFilters(asset, filters))
    .map(asset => enrichAssetData(asset)) // Expensive transformation
    .sort((a, b) => compareAssets(a, b)); // Expensive comparison

  const aggregated = filtered.reduce((acc, asset) => {
    acc.totalValue += parseFloat(asset.balance) * asset.price;
    acc.byChain[asset.chainId] = (acc.byChain[asset.chainId] || 0) + asset.value;
    return acc;
  }, { totalValue: 0, byChain: {} });

  return (
    &lt;div&gt;
      &lt;TotalValue value={aggregated.totalValue} /&gt;
      {filtered.map(asset => &lt;AssetCard key={asset.id} asset={asset} /&gt;)}
    &lt;/div&gt;
  );
};
```

```typescript
✅ CORRECT: Memoize expensive computations
const AssetDashboard = ({ assets, filters }: AssetDashboardProps) => {
  // Memoize filtered and enriched assets
  const filtered = useMemo(() => {
    return assets
      .filter(asset => matchesFilters(asset, filters))
      .map(asset => enrichAssetData(asset))
      .sort((a, b) => compareAssets(a, b));
  }, [assets, filters]); // Only recompute when dependencies change

  // Memoize aggregated data
  const aggregated = useMemo(() => {
    return filtered.reduce((acc, asset) => {
      acc.totalValue += parseFloat(asset.balance) * asset.price;
      acc.byChain[asset.chainId] = (acc.byChain[asset.chainId] || 0) + asset.value;
      return acc;
    }, { totalValue: 0, byChain: {} });
  }, [filtered]); // Depends on filtered, which is already memoized

  return (
    &lt;div&gt;
      &lt;TotalValue value={aggregated.totalValue} /&gt;
      {filtered.map(asset => &lt;AssetCard key={asset.id} asset={asset} /&gt;)}
    &lt;/div&gt;
  );
};
```

### Strategy: Move Complex Computations to Selectors

For Redux state, move expensive computations to selectors instead of components.

```typescript
❌ WRONG: Computation in component
const AssetList = () => {
  const assets = useSelector(state => state.assets);
  const filters = useSelector(state => state.filters);

  // Expensive computation runs in component render
  const filtered = assets
    .filter(asset => matchesFilters(asset, filters))
    .map(asset => expensiveTransform(asset));

  return &lt;div&gt;{filtered.map(a => &lt;Asset key={a.id} asset={a} /&gt;)}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: Computation in selector
// In selectors file:
const selectAssets = (state) => state.assets;
const selectFilters = (state) => state.filters;

const selectFilteredAssets = createSelector(
  [selectAssets, selectFilters],
  (assets, filters) => {
    // Only recomputes when assets or filters change
    return assets
      .filter(asset => matchesFilters(asset, filters))
      .map(asset => expensiveTransform(asset));
  },
);

// In component:
const AssetList = () => {
  // Selector handles memoization automatically
  const filteredAssets = useSelector(selectFilteredAssets);

  return &lt;div&gt;{filteredAssets.map(a => &lt;Asset key={a.id} asset={a} /&gt;)}&lt;/div&gt;;
};
```

### Strategy: Use Web Workers for Heavy Computations

For very expensive computations (crypto operations, large data transformations), use Web Workers.

```typescript
✅ GOOD: Offload heavy computation to Web Worker
// worker.ts
self.onmessage = (e) => {
  const { assets, filters } = e.data;

  // Heavy computation in worker thread
  const result = assets
    .filter(asset => matchesFilters(asset, filters))
    .map(asset => expensiveCryptoOperation(asset))
    .sort((a, b) => compareAssets(a, b));

  self.postMessage(result);
};

// Component
const AssetList = ({ assets, filters }: AssetListProps) => {
  const [processed, setProcessed] = useState([]);
  const workerRef = useRef&lt;Worker | null&gt;(null);

  useEffect(() => {
    workerRef.current = new Worker(new URL('./worker.ts', import.meta.url));

    workerRef.current.onmessage = (e) => {
      setProcessed(e.data);
    };

    return () => {
      workerRef.current?.terminate();
    };
  }, []);

  useEffect(() => {
    if (workerRef.current) {
      workerRef.current.postMessage({ assets, filters });
    }
  }, [assets, filters]);

  return &lt;div&gt;{processed.map(a => &lt;Asset key={a.id} asset={a} /&gt;)}&lt;/div&gt;;
};
```

### Strategy: Debounce Frequent Updates

```typescript
✅ GOOD: Debounce rapid balance updates
import { useDebouncedValue } from './hooks/useDebouncedValue';

const AssetBalance = ({ assetId }: AssetBalanceProps) => {
  const balance = useSelector(state => selectAssetBalance(state, assetId));

  // Debounce rapid updates to avoid jitter
  const debouncedBalance = useDebouncedValue(balance, 300);

  return &lt;div&gt;{debouncedBalance}&lt;/div&gt;;
};

// Hook implementation
function useDebouncedValue&lt;T&gt;(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debounced;
}
```

---

---

## Hook Optimizations

> **Tracking:** [Epic #6525: useEffect Anti-Patterns](https://github.com/MetaMask/MetaMask-planning/issues/6525)

### Don't Overuse useEffect

Many operations don't need effects. See: [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)

```typescript
❌ WRONG: Using effect for derived state
const TokenDisplay = ({ token }: TokenDisplayProps) => {
  const [displayName, setDisplayName] = useState('');

  useEffect(() => {
    setDisplayName(`${token.symbol} (${token.name})`);
  }, [token]);

  return &lt;div&gt;{displayName}&lt;/div&gt;;
};

✅ CORRECT: Calculate during render
const TokenDisplay = ({ token }: TokenDisplayProps) => {
  const displayName = `${token.symbol} (${token.name})`;
  return &lt;div&gt;{displayName}&lt;/div&gt;;
};
```

### Minimize useEffect Dependencies

```typescript
❌ WRONG: Too many dependencies
const TokenBalance = ({ address, network, refreshInterval }: Props) => {
  const [balance, setBalance] = useState('0');

  useEffect(() => {
    const fetch = async () => {
      const result = await fetchBalance(address, network);
      setBalance(result);
    };

    fetch();
    const interval = setInterval(fetch, refreshInterval);
    return () => clearInterval(interval);
  }, [address, network, refreshInterval]); // Effect runs too often

  return &lt;div&gt;{balance}&lt;/div&gt;;
};

✅ CORRECT: Reduce dependencies
const TokenBalance = ({ address, network, refreshInterval = 10000 }: Props) => {
  const [balance, setBalance] = useState('0');

  useEffect(() => {
    const fetch = async () => {
      const result = await fetchBalance(address, network);
      setBalance(result);
    };

    fetch();
    const interval = setInterval(fetch, refreshInterval);
    return () => clearInterval(interval);
  }, [address, network]); // refreshInterval moved to default param

  return &lt;div&gt;{balance}&lt;/div&gt;;
};
```

### 🔴 CRITICAL: JSON.stringify in useEffect Dependencies

> **Priority:** P0 - Critical Performance Impact
> **Tracking:** [Ticket #6545](https://github.com/MetaMask/MetaMask-planning/issues/6545)

Using `JSON.stringify` in dependencies is expensive and defeats memoization. However, there are valid use cases where you need to trigger effects when nested object properties change (deep equality) but not when only the object reference changes.

**The Problem:**

- `JSON.stringify` executes on every render, even if object hasn't changed
- String comparison is slower than reference comparison
- Creates new string objects, defeating memoization benefits
- Can cause infinite loops if stringified value changes reference
- Doesn't handle circular references or functions

```typescript
❌ WRONG: JSON.stringify runs on every render
const usePolling = (input: PollingInput) => {
  useEffect(() => {
    startPolling(input);
  }, [input &amp;&amp; JSON.stringify(input)]); // Expensive! Runs every render
};

// ❌ WRONG: useMemo with JSON.stringify defeats purpose
const jsonAccounts = useMemo(() => JSON.stringify(accounts), [accounts]);
```

**When You Need Deep Equality:**

You want to trigger effects when:

- ✅ Nested properties of an object change (deep equality)
- ✅ Array elements change (deep equality)
- ❌ Object reference changes but values are the same (should NOT trigger)

**Best Practices for Deep Equality Dependencies:**

```typescript
✅ CORRECT: Option 1 - Use useEqualityCheck hook (Recommended)
import { useEqualityCheck } from './hooks/useEqualityCheck';
import { isEqual } from 'lodash';

// useEqualityCheck returns a stable reference that only changes on deep equality
const usePolling = (input: PollingInput) => {
  // Returns same reference if deep values are equal, new reference if different
  const stableInput = useEqualityCheck(input, isEqual);

  useEffect(() => {
    startPolling(stableInput);
  }, [stableInput]); // Only triggers when deep values actually change
};

// Example: Network configuration object
const TokenBalance = ({ networkConfig }: TokenBalanceProps) => {
  // networkConfig may get new reference on every render, but values rarely change
  const stableConfig = useEqualityCheck(networkConfig, isEqual);

  useEffect(() => {
    fetchBalance(stableConfig);
  }, [stableConfig]); // Only re-fetches when config values actually change
};
```

```typescript
✅ CORRECT: Option 2 - useRef with deep equality check in effect
import { isEqual } from 'lodash';

const usePolling = (input: PollingInput) => {
  const inputRef = useRef(input);

  useEffect(() => {
    // Only execute if deep values changed
    if (!isEqual(input, inputRef.current)) {
      inputRef.current = input;
      startPolling(input);
    }
  }, [input]); // Effect runs on every input change, but logic only executes on deep equality
};
```

```typescript
✅ CORRECT: Option 3 - Normalize to stable primitives (Best for Redux)
// If possible, extract stable primitive values instead of objects
const usePolling = (input: PollingInput) => {
  // Extract only the values that actually matter
  const inputId = useMemo(() => input.id, [input.id]);
  const inputChainId = useMemo(() => input.chainId, [input.chainId]);
  const inputRpcUrl = useMemo(() => input.rpcUrl, [input.rpcUrl]);

  useEffect(() => {
    startPolling(input);
  }, [inputId, inputChainId, inputRpcUrl]); // Only depends on primitives
};

// For Redux: Use selectors that return stable references
const selectNetworkConfig = createDeepEqualSelector(
  [(state) => state.network],
  (network) => ({
    chainId: network.chainId,
    rpcUrl: network.rpcUrl,
  })
);

const TokenBalance = () => {
  const networkConfig = useSelector(selectNetworkConfig); // Stable reference
  useEffect(() => {
    fetchBalance(networkConfig);
  }, [networkConfig]); // Only triggers when config values change
};
```

**When to Use Each Approach:**

| Approach                    | Use When                                    | Pros                                 | Cons                  |
| --------------------------- | ------------------------------------------- | ------------------------------------ | --------------------- |
| **useEqualityCheck**        | Objects/arrays from props or external state | Simple, reusable, handles edge cases | Requires hook import  |
| **useRef + isEqual**        | One-off cases, custom logic needed          | Full control, no extra hook          | More boilerplate      |
| **Normalize to primitives** | Can extract stable IDs/values               | Most performant, clear dependencies  | Not always possible   |

**Key Principles:**

1. ✅ **Use deep equality when object references change frequently but values don't** - Common with Redux selectors, props from parent components, or API responses
2. ✅ **Prefer `useEqualityCheck` hook** - Already implemented in codebase, handles edge cases
3. ✅ **Normalize when possible** - Extract stable primitives (IDs, strings, numbers) instead of objects
4. ❌ **Never use `JSON.stringify`** - Expensive, unreliable, breaks with functions/circular refs
5. ❌ **Don't skip dependencies** - Always include dependencies, use deep equality to stabilize them

### ❌ Anti-Pattern: useEffect with Incomplete Dependencies (Stale Closures)

Empty dependency arrays that use values from closure create stale closures.

```typescript
❌ WRONG: Empty deps but uses values from closure
const Name = ({ type, name }: NameProps) => {
  useEffect(() => {
    trackEvent({
      properties: {
        petname_category: type,  // Uses 'type' from closure
        has_petname: Boolean(name?.length),  // Uses 'name' from closure
      },
    });
  }, []); // Empty deps - 'type' and 'name' are stale!
};
```

**Problems:**

- Values captured in closure may be stale (initial values, not current)
- Effect runs once but uses outdated values
- Can lead to incorrect analytics/metrics
- Hard to debug because values appear correct in code

```typescript
✅ CORRECT: Include all dependencies
const Name = ({ type, name }: NameProps) => {
  useEffect(() => {
    trackEvent({
      properties: {
        petname_category: type,
        has_petname: Boolean(name?.length),
      },
    });
  }, [type, name]); // Include all dependencies

  // OR if you truly only want to track once:
  const hasTrackedRef = useRef(false);
  useEffect(() => {
    if (!hasTrackedRef.current) {
      trackEvent({
        properties: {
          petname_category: type,
          has_petname: Boolean(name?.length),
        },
      });
      hasTrackedRef.current = true;
    }
  }, [type, name]);
};
```

### ❌ Anti-Pattern: Wrong Dependencies in useMemo/useCallback

Missing dependencies in `useMemo`/`useCallback` cause stale closures and bugs.

```typescript
❌ WRONG: Missing dependencies
const TokenList = ({ tokens, filter }: TokenListProps) => {
  // Dependencies are wrong - should include filter!
  const filteredTokens = useMemo(() => {
    return tokens.filter(token => token.symbol.includes(filter));
  }, [tokens]); // Missing filter dependency!

  return &lt;div&gt;...&lt;/div&gt;;
};
```

**Problems:**

- Stale closures capture old values
- Effects/calculations use outdated data
- Hard to debug because code looks correct
- Can cause incorrect behavior or infinite loops

**Solution:** Always include all dependencies. Use ESLint rule `react-hooks/exhaustive-deps` to catch missing dependencies automatically.

```typescript
✅ CORRECT: Include all dependencies
const TokenList = ({ tokens, filter }: TokenListProps) => {
  const filteredTokens = useMemo(() => {
    return tokens.filter(token => token.symbol.includes(filter));
  }, [tokens, filter]); // All dependencies included

  return &lt;div&gt;...&lt;/div&gt;;
};
```

### ❌ Anti-Pattern: Cascading useEffect Chains

Multiple effects where one sets state that triggers another cause unnecessary re-renders.

```typescript
❌ WRONG: Effect chain - first effect sets state, second responds to it
const useHistoricalPrices = () => {
  const [prices, setPrices] = useState([]);
  const [metadata, setMetadata] = useState(null);

  // First effect fetches and updates Redux
  useEffect(() => {
    fetchPrices();
    const intervalId = setInterval(fetchPrices, 60000);
    return () => clearInterval(intervalId);
  }, [chainId, address]);

  // Second effect responds to Redux state change
  useEffect(() => {
    const pricesToSet = historicalPricesNonEvm?.[address]?.intervals ?? [];
    setPrices(pricesToSet); // Triggers third effect
  }, [historicalPricesNonEvm, address]);

  // Third effect depends on state from second effect
  useEffect(() => {
    const metadataToSet = deriveMetadata(prices);
    setMetadata(metadataToSet);
  }, [prices]);
};
```

**Problems:**

- Multiple re-renders from cascading effects
- Hard to reason about execution order
- Can cause race conditions
- Performance overhead from multiple effect runs

```typescript
✅ CORRECT: Combine effects or compute during render
const useHistoricalPrices = () => {
  // Compute prices during render from Redux state
  const prices = useMemo(() => {
    return historicalPricesNonEvm?.[address]?.intervals ?? [];
  }, [historicalPricesNonEvm, address]);

  // Compute metadata during render from prices
  const metadata = useMemo(() => {
    return deriveMetadata(prices);
  }, [prices]);

  // Single effect for async operations
  useEffect(() => {
    fetchPrices();
    const intervalId = setInterval(fetchPrices, 60000);
    return () => clearInterval(intervalId);
  }, [chainId, address]);

  return { prices, metadata };
};
```

### ❌ Anti-Pattern: Conditional Early Return with All Dependencies

Effects with conditional early returns that still include all dependencies in the array.

```typescript
❌ WRONG: Early return but dependencies include unused values
const useHistoricalPrices = ({ isEvm, chainId, address }: Props) => {
  useEffect(() => {
    if (isEvm) {
      return; // Early return
    }
    // Only uses chainId and address when not EVM
    fetchPrices(chainId, address);
  }, [isEvm, chainId, address]); // Includes all deps even when unused
};
```

**Problems:**

- Effect dependencies include values not used when condition is true
- Can cause unnecessary effect re-runs
- Unclear intent
- ESLint may warn about missing dependencies

```typescript
✅ CORRECT: Split into separate effects or use proper conditional logic
// Option 1: Split effects
const useHistoricalPrices = ({ isEvm, chainId, address }: Props) => {
  useEffect(() => {
    if (!isEvm) {
      fetchPrices(chainId, address);
    }
  }, [isEvm, chainId, address]); // All deps are used

  // OR Option 2: Separate effects
  useEffect(() => {
    if (isEvm) return;
    fetchPrices(chainId, address);
  }, [isEvm]); // Only depends on condition

  useEffect(() => {
    if (!isEvm) {
      fetchPrices(chainId, address);
    }
  }, [chainId, address]); // Only when not EVM
};
```

---

### ❌ Anti-Pattern: Regular Variables Instead of useRef

Using regular variables instead of `useRef` for values that need to persist across renders.

```typescript
❌ WRONG: Regular variable gets reset on every render
const usePolling = (input: PollingInput) => {
  let isMounted = false; // Gets reset every render!

  useEffect(() => {
    isMounted = true;
    startPolling(input);

    return () => {
      isMounted = false;
    };
  }, [input]);
};
```

**Problems:**

- Regular variable gets reset on every render
- Doesn't persist across renders
- Closure captures stale value
- Can cause bugs with async operations

```typescript
✅ CORRECT: Use useRef for persistent values
const usePolling = (input: PollingInput) => {
  const isMountedRef = useRef(false);

  useEffect(() => {
    isMountedRef.current = true;
    startPolling(input);

    return () => {
      isMountedRef.current = false;
    };
  }, [input]);
};
```

## Non-Deterministic Hook Execution

### ❌ Anti-Pattern: Conditional Hook Calls

Hooks must be called in the same order on every render. Conditional hooks cause bugs and performance issues.

```typescript
❌ WRONG: Conditional hook execution
const TokenDisplay = ({ token, showDetails }: TokenDisplayProps) => {
  const [balance, setBalance] = useState('0');

  if (showDetails) {
    // ⚠️ Hook called conditionally - breaks Rules of Hooks!
    const [metadata, setMetadata] = useState(null);
    useEffect(() => {
      fetchMetadata(token.id).then(setMetadata);
    }, [token.id]);
  }

  return &lt;div&gt;{balance}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: Always call hooks unconditionally
const TokenDisplay = ({ token, showDetails }: TokenDisplayProps) => {
  const [balance, setBalance] = useState('0');
  const [metadata, setMetadata] = useState(null);

  useEffect(() => {
    if (showDetails) {
      fetchMetadata(token.id).then(setMetadata);
    }
  }, [token.id, showDetails]);

  return (
    &lt;div&gt;
      &lt;div&gt;Balance: {balance}&lt;/div&gt;
      {showDetails &amp;&amp; metadata &amp;&amp; &lt;div&gt;Metadata: {metadata.name}&lt;/div&gt;}
    &lt;/div&gt;
  );
};
```

### ❌ Anti-Pattern: Dynamic Hook Creation

Creating hooks dynamically or in loops breaks React's hook order tracking.

```typescript
❌ WRONG: Dynamic hook creation
const AssetList = ({ assets }: AssetListProps) => {
  // ⚠️ Number of hooks changes based on assets.length!
  const balances = assets.map(asset => {
    const [balance, setBalance] = useState('0'); // Wrong!
    useEffect(() => {
      fetchBalance(asset.id).then(setBalance);
    }, [asset.id]);
    return balance;
  });

  return &lt;div&gt;{balances.map((b, i) => &lt;div key={i}&gt;{b}&lt;/div&gt;)}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: Use custom hook or component
// Option 1: Custom hook for single asset
const useAssetBalance = (assetId: string) => {
  const [balance, setBalance] = useState('0');

  useEffect(() => {
    fetchBalance(assetId).then(setBalance);
  }, [assetId]);

  return balance;
};

const AssetList = ({ assets }: AssetListProps) => {
  return (
    &lt;div&gt;
      {assets.map(asset => (
        &lt;AssetItem key={asset.id} asset={asset} /&gt;
      ))}
    &lt;/div&gt;
  );
};

// Option 2: Component with its own hooks
const AssetItem = ({ asset }: { asset: Asset }) => {
  const balance = useAssetBalance(asset.id);
  return &lt;div&gt;{asset.name}: {balance}&lt;/div&gt;;
};
```

## Cascading Re-renders from Hook Dependencies

### ❌ Anti-Pattern: Chain Reaction Re-renders

When hooks depend on values that change frequently, they can cause cascading re-renders.

```typescript
❌ WRONG: Cascading re-renders
const Dashboard = () => {
  const accounts = useSelector(state => state.accounts); // Large array
  const [filteredAccounts, setFilteredAccounts] = useState([]);

  // Effect runs whenever accounts array reference changes
  useEffect(() => {
    const filtered = accounts.filter(a => a.isActive);
    setFilteredAccounts(filtered); // Triggers re-render
  }, [accounts]); // accounts reference changes frequently

  // Another effect depends on filteredAccounts
  useEffect(() => {
    updateAnalytics(filteredAccounts); // Triggers another update
  }, [filteredAccounts]);

  return &lt;div&gt;{filteredAccounts.map(a => &lt;Account key={a.id} account={a} /&gt;)}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: Use selectors and memoization to break chain
// In selectors file:
const selectAccounts = (state) => state.accounts;
const selectActiveAccounts = createSelector(
  [selectAccounts],
  (accounts) => accounts.filter(a => a.isActive),
);

// In component:
const Dashboard = () => {
  // Selector handles memoization - only changes when accounts actually change
  const activeAccounts = useSelector(selectActiveAccounts);

  // Memoize analytics update to prevent unnecessary calls
  const analyticsRef = useRef(activeAccounts);
  useEffect(() => {
    if (analyticsRef.current !== activeAccounts) {
      updateAnalytics(activeAccounts);
      analyticsRef.current = activeAccounts;
    }
  }, [activeAccounts]);

  return &lt;div&gt;{activeAccounts.map(a => &lt;Account key={a.id} account={a} /&gt;)}&lt;/div&gt;;
};
```

### Strategy: Isolate Hook Dependencies

```typescript
❌ WRONG: Hook depends on frequently changing object
const TokenCard = ({ token }: TokenCardProps) => {
  const [formattedBalance, setFormattedBalance] = useState('');

  useEffect(() => {
    // token object reference changes frequently
    setFormattedBalance(formatBalance(token.balance, token.decimals));
  }, [token]); // Re-runs too often

  return &lt;div&gt;{formattedBalance}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: Extract stable values, use refs for callbacks
const TokenCard = ({ token }: TokenCardProps) => {
  // Extract primitive values that change less frequently
  const balance = token.balance;
  const decimals = token.decimals;

  // Calculate during render instead of effect
  const formattedBalance = useMemo(
    () => formatBalance(balance, decimals),
    [balance, decimals]
  );

  return &lt;div&gt;{formattedBalance}&lt;/div&gt;;
};
```

### Use Component Composition to Prevent Re-renders

```typescript
❌ WRONG: Children re-render when parent state changes
const Dashboard = () => {
  const [count, setCount] = useState(0);

  return (
    &lt;div&gt;
      &lt;button onClick={() => setCount(c => c + 1)}&gt;
        Count: {count}
      &lt;/button&gt;
      &lt;ExpensiveChart /&gt; {/* Re-renders unnecessarily! */}
      &lt;ExpensiveTable /&gt; {/* Re-renders unnecessarily! */}
    &lt;/div&gt;
  );
};

✅ CORRECT: Move state down
const Dashboard = () => {
  return (
    &lt;div&gt;
      &lt;Counter /&gt; {/* Only this re-renders */}
      &lt;ExpensiveChart /&gt;
      &lt;ExpensiveTable /&gt;
    &lt;/div&gt;
  );
};

const Counter = () => {
  const [count, setCount] = useState(0);
  return (
    &lt;button onClick={() => setCount(c => c + 1)}&gt;
      Count: {count}
    &lt;/button&gt;
  );
};

✅ ALSO CORRECT: Pass children as props
const Dashboard = ({ children }: { children: React.ReactNode }) => {
  const [count, setCount] = useState(0);

  return (
    &lt;div&gt;
      &lt;button onClick={() => setCount(c => c + 1)}&gt;
        Count: {count}
      &lt;/button&gt;
      {children} {/* Children don't re-render! */}
    &lt;/div&gt;
  );
};

// Usage:
&lt;Dashboard&gt;
  &lt;ExpensiveChart /&gt;
  &lt;ExpensiveTable /&gt;
&lt;/Dashboard&gt;
```

---

## Async Operations and Cleanup

### ❌ Anti-Pattern: State Updates After Component Unmount

Updating state after unmount causes memory leaks and React warnings.

```typescript
❌ WRONG: No mounted check
const TokenBalance = ({ address }: TokenBalanceProps) => {
  const [balance, setBalance] = useState('0');

  useEffect(() => {
    fetchBalance(address).then(result => {
      setBalance(result); // ⚠️ May update after unmount!
    });
  }, [address]);

  return &lt;div&gt;{balance}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: Check mounted state before updating
const TokenBalance = ({ address }: TokenBalanceProps) => {
  const [balance, setBalance] = useState('0');

  useEffect(() => {
    let cancelled = false;

    const fetch = async () => {
      const result = await fetchBalance(address);
      if (!cancelled) {
        setBalance(result);
      }
    };

    fetch();

    return () => {
      cancelled = true;
    };
  }, [address]);

  return &lt;div&gt;{balance}&lt;/div&gt;;
};
```

### ❌ Anti-Pattern: Missing AbortController for Fetch Requests

Fetch requests without abort signals continue running after unmount, wasting resources.

```typescript
❌ WRONG: No abort signal
const AssetList = ({ chainId }: AssetListProps) => {
  const [assets, setAssets] = useState([]);

  useEffect(() => {
    fetch(`/api/assets/${chainId}`)
      .then(res => res.json())
      .then(data => setAssets(data)); // Request continues after unmount!
  }, [chainId]);

  return &lt;div&gt;{assets.map(a => &lt;Asset key={a.id} asset={a} /&gt;)}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: AbortController for cleanup
const AssetList = ({ chainId }: AssetListProps) => {
  const [assets, setAssets] = useState([]);

  useEffect(() => {
    const controller = new AbortController();

    fetch(`/api/assets/${chainId}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => {
        if (!controller.signal.aborted) {
          setAssets(data);
        }
      })
      .catch(error => {
        if (error.name !== 'AbortError') {
          console.error('Failed to fetch assets:', error);
        }
      });

    return () => {
      controller.abort(); // Cancels request on unmount
    };
  }, [chainId]);

  return &lt;div&gt;{assets.map(a => &lt;Asset key={a.id} asset={a} /&gt;)}&lt;/div&gt;;
};
```

### ❌ Anti-Pattern: Missing Cleanup for Intervals/Subscriptions

Timers and subscriptions must be cleaned up to prevent memory leaks.

```typescript
❌ WRONG: Interval never cleared
const PriceTicker = ({ tokenAddress }: PriceTickerProps) => {
  const [price, setPrice] = useState(0);

  useEffect(() => {
    const interval = setInterval(async () => {
      const newPrice = await fetchPrice(tokenAddress);
      setPrice(newPrice);
    }, 1000); // ⚠️ Interval continues after unmount!

    // Missing cleanup!
  }, [tokenAddress]);

  return &lt;div&gt;${price}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: Cleanup interval
const PriceTicker = ({ tokenAddress }: PriceTickerProps) => {
  const [price, setPrice] = useState(0);

  useEffect(() => {
    let cancelled = false;

    const fetchPrice = async () => {
      const newPrice = await fetchPriceData(tokenAddress);
      if (!cancelled) {
        setPrice(newPrice);
      }
    };

    fetchPrice(); // Initial fetch
    const interval = setInterval(fetchPrice, 1000);

    return () => {
      cancelled = true;
      clearInterval(interval); // Cleanup on unmount
    };
  }, [tokenAddress]);

  return &lt;div&gt;${price}&lt;/div&gt;;
};
```

### ❌ Anti-Pattern: Large Object Retention in Closures

Closures capturing large objects prevent garbage collection.

```typescript
❌ WRONG: Large object retained in closure
const TransactionList = ({ transactions }: TransactionListProps) => {
  const [filtered, setFiltered] = useState([]);

  useEffect(() => {
    // Large transactions array captured in closure
    const expensiveFilter = () => {
      return transactions
        .filter(tx => tx.status === 'pending')
        .map(tx => expensiveTransform(tx)); // Large object retained!
    };

    const interval = setInterval(() => {
      setFiltered(expensiveFilter());
    }, 5000);

    return () => clearInterval(interval);
  }, [transactions]); // transactions array reference changes frequently

  return &lt;div&gt;{filtered.map(tx => &lt;Transaction key={tx.id} tx={tx} /&gt;)}&lt;/div&gt;;
};
```

```typescript
✅ CORRECT: Extract only needed data, use refs for stable references
const TransactionList = ({ transactions }: TransactionListProps) => {
  const [filtered, setFiltered] = useState([]);
  const transactionsRef = useRef(transactions);

  // Update ref without causing effect to re-run
  useEffect(() => {
    transactionsRef.current = transactions;
  }, [transactions]);

  useEffect(() => {
    let cancelled = false;

    const expensiveFilter = () => {
      // Use ref to avoid capturing transactions in closure
      const currentTransactions = transactionsRef.current;
      return currentTransactions
        .filter(tx => tx.status === 'pending')
        .map(tx => ({
          id: tx.id,
          amount: tx.amount,
          // Only extract needed properties, not entire object
        }));
    };

    const updateFiltered = () => {
      if (!cancelled) {
        setFiltered(expensiveFilter());
      }
    };

    updateFiltered();
    const interval = setInterval(updateFiltered, 5000);

    return () => {
      cancelled = true;
      clearInterval(interval);
    };
  }, []); // Empty deps - uses ref instead

  return &lt;div&gt;{filtered.map(tx => &lt;Transaction key={tx.id} tx={tx} /&gt;)}&lt;/div&gt;;
};
```

---

---

## 🔴 CRITICAL: Never Use useSelector with isEqual Comparator

> **Priority:** P0 - Critical Performance Impact  
> **Tracking:** [Ticket #6536](https://github.com/MetaMask/MetaMask-planning/issues/6536)

**This is one of the most impactful anti-patterns in the codebase.** The `isEqual` function passed to `useSelector` executes on **every component render**, not just when state changes.

### The Problem

```typescript
❌ CRITICAL: isEqual runs on EVERY render
const MyComponent = () => {
  // This deep comparison runs 100+ times/sec during quote refresh
  const bridgeQuotes = useSelector(getBridgeQuotes, isEqual);
  const quoteRequest = useSelector(getQuoteRequest, isEqual);
  const fromChain = useSelector(getFromChain, isEqual);
  // ...
};
```

**Impact:**
- 59 occurrences × 100 renders/sec = **5,900+ deep object traversals/sec**
- Each `isEqual` traverses the entire object tree
- Blocks main thread during swap/bridge quote flows
- Creates INP spikes of 200-400ms

### The Fix

Fix the underlying selector to return stable references:

```typescript
✅ CORRECT: Fix the selector instead
// Option 1: Use createDeepEqualSelector in selector definition
export const getBridgeQuotes = createDeepEqualSelector(
  [selectBridgeState],
  (bridgeState) => bridgeState.quotes,
);

// Option 2: Return primitives instead of objects
export const getFromChainId = createSelector(
  [selectBridgeState],
  (bridgeState) => bridgeState.fromChain?.chainId, // Primitive = stable
);

// In component - NO isEqual needed
const MyComponent = () => {
  const bridgeQuotes = useSelector(getBridgeQuotes); // ✅ Clean
  const fromChainId = useSelector(getFromChainId);   // ✅ Clean
};
```

### Migration Strategy

1. **Audit** all `useSelector(..., isEqual)` calls
2. **For each**, determine why `isEqual` was added (selector returns unstable ref)
3. **Fix the selector** using `createDeepEqualSelector` or return primitives
4. **Remove** the `isEqual` argument from `useSelector`
5. **Test** with React DevTools Profiler to verify no re-render regression

---

## Advanced Selector Patterns

> **Tracking:** [Epic #6524: Selector Optimization](https://github.com/MetaMask/MetaMask-planning/issues/6524)

### ❌ Anti-Pattern: Identity Functions as Output Selectors

Identity functions in `createSelector` provide no memoization benefit and waste memory.

```typescript
❌ WRONG: Identity function provides no memoization
export const getInternalAccounts = createSelector(
  (state: AccountsState) =>
    Object.values(state.metamask.internalAccounts.accounts),
  (accounts) => accounts, // Identity function - no transformation!
);

// This selector always returns a new array reference even if accounts haven't changed
// because Object.values() creates a new array every time
```

**Problems:**

- `Object.values()` creates a new array reference on every call
- Identity function doesn't prevent recalculation
- Downstream selectors re-run unnecessarily
- Memory waste from unnecessary array creation

```typescript
✅ CORRECT: Proper memoization with stable reference
export const getInternalAccounts = createSelector(
  (state: AccountsState) => state.metamask.internalAccounts.accounts,
  (accountsObject) => {
    // Only create array when accountsObject actually changes
    const accounts = Object.values(accountsObject);
    return accounts;
  },
);

// OR: Use createDeepEqualSelector if you need deep equality
export const getInternalAccounts = createDeepEqualSelector(
  (state: AccountsState) => state.metamask.internalAccounts.accounts,
  (accountsObject) => Object.values(accountsObject),
);
```

### ❌ Anti-Pattern: Selectors That Return Entire State Objects

Selectors that return large state objects cause unnecessary re-renders.

```typescript
❌ WRONG: Returns entire state slice
const selectAccountTreeStateForBalances = createSelector(
  (state: BalanceCalculationState) => state.metamask,
  (metamaskState) => metamaskState, // Returns entire metamask state!
);

// Every component using this selector re-renders on ANY metamask state change
```

```typescript
✅ CORRECT: Select only needed properties
const selectAccountTreeStateForBalances = createSelector(
  [
    (state: BalanceCalculationState) => state.metamask.accountTree,
    (state: BalanceCalculationState) => state.metamask.accountGroupsMetadata,
    (state: BalanceCalculationState) => state.metamask.accountWalletsMetadata,
  ],
  (accountTree, accountGroupsMetadata, accountWalletsMetadata) => ({
    accountTree: accountTree ?? EMPTY_ACCOUNT_TREE,
    accountGroupsMetadata: accountGroupsMetadata ?? EMPTY_OBJECT,
    accountWalletsMetadata: accountWalletsMetadata ?? EMPTY_OBJECT,
  }),
);
```

### ❌ Anti-Pattern: Selectors Without Proper Input Selectors

Selectors that access state directly instead of using input selectors can't be composed efficiently.

```typescript
❌ WRONG: Direct state access, can't be composed
const selectExpensiveComputation = createSelector(
  (state) => state.metamask, // Too broad
  (metamask) => {
    // Expensive computation using many properties
    return metamask.tokens
      .filter(t => t.balance > 0)
      .map(t => ({ ...t, computed: expensiveTransform(t) }))
      .sort((a, b) => b.balance - a.balance);
  },
);

✅ CORRECT: Granular input selectors for composition
const selectTokens = (state) => state.metamask.tokens;
const selectTokenBalances = (state) => state.metamask.tokenBalances;

const selectExpensiveComputation = createSelector(
  [selectTokens, selectTokenBalances],
  (tokens, balances) => {
    // Only recomputes when tokens or balances change
    return tokens
      .filter(t => balances[t.address] > 0)
      .map(t => ({ ...t, computed: expensiveTransform(t) }))
      .sort((a, b) => balances[b.address] - balances[a.address]);
  },
);
```

### Best Practices for Reselect Selectors

1. **Use granular input selectors** - Select smallest possible state slices
2. **Avoid identity functions** - Always transform data in output selector
3. **Reach for `createDeepEqualSelector` only when shallow checks fail** - Prefer `createSelector` by default; use deep equality when inputs keep the same reference but nested values change (see guidance below)
4. **Compose selectors** - Build complex selectors from simple ones
5. **Normalize state shape** - Use `byId` patterns to avoid deep nesting

```typescript
✅ GOOD: Well-structured selector composition
// Base selectors (granular)
const selectAccountsObject = (state) => state.metamask.internalAccounts.accounts;
const selectSelectedAccountId = (state) => state.metamask.internalAccounts.selectedAccount;

// Composed selector
const selectSelectedAccount = createSelector(
  [selectAccountsObject, selectSelectedAccountId],
  (accounts, selectedId) => accounts[selectedId] ?? null,
);

// Derived selector (reuses base selectors)
const selectSelectedAccountAddress = createSelector(
  [selectSelectedAccount],
  (account) => account?.address ?? null,
);
```

### Use `createDeepEqualSelector` sparingly

> **Tracking:** [Ticket #6537](https://github.com/MetaMask/MetaMask-planning/issues/6537)

`updateMetamaskState` applies background patches to Redux using Immer. Each call receives an array of Immer patches from the background controllers, runs them through `applyPatches`, then dispatches the resulting state.

Immer guarantees structural sharing: only the objects along the mutated path receive new references, while untouched branches retain their identity. The implications for selector memoization are:

- **`createSelector` (shallow equality on inputs)**
  - Works best when input selectors point directly at the branch that changes. When Immer patches adjust a deep property, the parent objects along that path get new references, so selectors such as `state => state.metamask.internalAccounts.accounts` see the change and recompute.
  - If a selector uses a very broad input (for example `state => state.metamask`), Immer still returns a brand-new `metamask` object on every patch, so the selector recomputes even when the actual slice it cares about did not change. Keep inputs tight to avoid unnecessary work.
- **`createDeepEqualSelector` (deep equality on inputs)**
  - Because it compares arguments with `lodash/isEqual`, a selector can ignore the fact that Immer provided a fresh reference when the underlying data is unchanged. This helps when patches touch other controllers but Redux still replaces the parent object you depend on.
  - The trade-off is that `isEqual` runs on every evaluation, which is noticeable for large nested payloads. Only use the deep-equality variant when you have evidence that Immer's structural sharing is still producing noisy reference changes for your selector's inputs.

In practice:

- For selectors whose inputs come from a store that mutates nested properties without replacing the top-level reference (for example, background controllers that update metadata in place), `createSelector` is sufficient and cheaper.
- For selectors rebuilding a complex aggregate (sorting, merging, normalizing) on every call even though the semantic contents often stay the same (for example `getWalletsWithAccounts`), `createDeepEqualSelector` avoids downstream re-renders by tolerating reference churn.
- Regardless of memoization strategy, keep input selectors granular so Immer's minimal reference changes flow only to the consumers that truly need to update.

**Guard rails:**

- If a deep selector becomes hot, profile it with React DevTools before shipping.
- Document why you chose `createDeepEqualSelector` so future contributors can revisit the trade-off.

### Combine related selectors into one memoized selector

Each `useSelector` subscription runs independently. When a component calls multiple selectors in sequence, Redux evaluates each one and triggers rerenders whenever their results change—even if they depend on the same underlying state. Consolidating those calls into a single memoized selector reduces redundant work and keeps derived data co-located.

```typescript
❌ WRONG: 11+ separate useSelector calls
const {
  activeQuote,
  isQuoteGoingToRefresh,
  isLoading: isQuoteLoading,
} = useSelector(getBridgeQuotes);
const currency = useSelector(getCurrentCurrency);
const { insufficientBal } = useSelector(getQuoteRequest);
const fromChain = useSelector(getFromChain);
const locale = useSelector(getIntlLocale);
const isStxEnabled = useSelector(getIsStxEnabled);
const fromToken = useSelector(getFromToken);
const toToken = useSelector(getToToken);
const slippage = useSelector(getSlippage);
const isSolanaSwap = useSelector(getIsSolanaSwap);
const isToOrFromNonEvm = useSelector(getIsToOrFromNonEvm);
const priceImpactThresholds = useSelector(getPriceImpactThresholds);
```

This pattern subscribes the component 11 different times. If any selector emits a new reference, React schedules a rerender—even when the fields you care about stay the same.

```typescript
✅ CORRECT: Single composed selector
const selectBridgeQuoteCardView = createSelector(
  [
    getBridgeQuotes,
    getCurrentCurrency,
    getQuoteRequest,
    getFromChain,
    getIntlLocale,
    getIsStxEnabled,
    getFromToken,
    getToToken,
    getSlippage,
    getIsSolanaSwap,
    getIsToOrFromNonEvm,
    getPriceImpactThresholds,
  ],
  (
    bridgeQuotes,
    currency,
    quoteRequest,
    fromChain,
    locale,
    isStxEnabled,
    fromToken,
    toToken,
    slippage,
    isSolanaSwap,
    isToOrFromNonEvm,
    priceImpactThresholds,
  ) => ({
    activeQuote: bridgeQuotes.activeQuote,
    isQuoteGoingToRefresh: bridgeQuotes.isQuoteGoingToRefresh,
    isQuoteLoading: bridgeQuotes.isLoading,
    currency,
    insufficientBal: quoteRequest.insufficientBal,
    fromChain,
    locale,
    isStxEnabled,
    fromToken,
    toToken,
    slippage,
    isSolanaSwap,
    isToOrFromNonEvm,
    priceImpactThresholds,
  }),
);

const MultichainBridgeQuoteCard = () => {
  const {
    activeQuote,
    isQuoteGoingToRefresh,
    isQuoteLoading,
    currency,
    insufficientBal,
    fromChain,
    locale,
    isStxEnabled,
    fromToken,
    toToken,
    slippage,
    isSolanaSwap,
    isToOrFromNonEvm,
    priceImpactThresholds,
  } = useSelector(selectBridgeQuoteCardView);
  // ...
};
```

**Benefits:**

- Only one subscription; the component rerenders once per state change instead of once per selector.
- Shared memoization ensures the combined output only changes when at least one dependency does.
- Centralizes domain-specific shaping logic in the selector layer, simplifying reuse and testing.
- Lets you switch to `createDeepEqualSelector` (when justified) in a single place if background patches mutate nested data without replacing references.

### ❌ Anti-Pattern: Inline Selector Functions in useSelector

Creating selector functions inline in `useSelector` breaks memoization and creates new references.

```typescript
❌ WRONG: Inline function creates new reference every render
const Connections = () => {
  const subjectMetadata = useSelector((state) => {
    return getConnectedSitesList(state);
  });

  const connectedAccountGroups = useSelector((state) => {
    if (!showConnectionStatus || permittedAddresses.length === 0) {
      return [];
    }
    return getAccountGroupsByAddress(state, permittedAddresses);
  });
};
```

**Problems:**

- New function reference on every render
- Redux can't optimize selector calls
- Breaks selector memoization
- Causes unnecessary subscriptions and re-renders

```typescript
✅ CORRECT: Extract to memoized selector or use useCallback
// Option 1: Extract to memoized selector (preferred)
const selectConnectedAccountGroups = createSelector(
  [
    (state) => state,
    (_state, showConnectionStatus: boolean) => showConnectionStatus,
    (_state, _showConnectionStatus, permittedAddresses: string[]) => permittedAddresses,
  ],
  (state, showConnectionStatus, permittedAddresses) => {
    if (!showConnectionStatus || permittedAddresses.length === 0) {
      return [];
    }
    return getAccountGroupsByAddress(state, permittedAddresses);
  },
);

const Connections = () => {
  const subjectMetadata = useSelector(getConnectedSitesList);
  const connectedAccountGroups = useSelector((state) =>
    selectConnectedAccountGroups(state, showConnectionStatus, permittedAddresses)
  );
};

// Option 2: Use useCallback for selector function
const Connections = () => {
  const selectConnectedGroups = useCallback(
    (state) => {
      if (!showConnectionStatus || permittedAddresses.length === 0) {
        return [];
      }
      return getAccountGroupsByAddress(state, permittedAddresses);
    },
    [showConnectionStatus, permittedAddresses]
  );

  const connectedAccountGroups = useSelector(selectConnectedGroups);
};
```

### ❌ Anti-Pattern: Multiple useSelector Calls Selecting Same State Slice

Multiple `useSelector` calls for the same state slice create unnecessary subscriptions.

```typescript
❌ WRONG: Multiple selectors for same state slice
const Routes = () => {
  const alertOpen = useAppSelector((state) => state.appState.alertOpen);
  const alertMessage = useAppSelector((state) => state.appState.alertMessage);
  const isLoading = useAppSelector((state) => state.appState.isLoading);
  const loadingMessage = useAppSelector((state) => state.appState.loadingMessage);
  // ... 20+ more selectors from same slice
};
```

**Problems:**

- Each `useSelector` creates a separate subscription
- Multiple subscriptions to the same state slice
- More overhead than selecting the whole slice once
- Can cause unnecessary re-renders

```typescript
✅ CORRECT: Select entire slice once or create single selector
// Option 1: Select entire slice once
const Routes = () => {
  const appState = useAppSelector((state) => state.appState);
  const { alertOpen, alertMessage, isLoading, loadingMessage } = appState;
};

// Option 2: Create single memoized selector
const selectAppState = (state) => state.appState;
const selectAppStateSlice = createSelector(
  [selectAppState],
  (appState) => ({
    alertOpen: appState.alertOpen,
    alertMessage: appState.alertMessage,
    isLoading: appState.isLoading,
    loadingMessage: appState.loadingMessage,
    // ... other properties
  })
);

const Routes = () => {
  const appStateSlice = useAppSelector(selectAppStateSlice);
};
```

### ❌ Anti-Pattern: Inefficient Use of `Object.values()` and `Object.keys()` in Selectors

> **Tracking:** [Ticket #6539](https://github.com/MetaMask/MetaMask-planning/issues/6539)

**Problem:** When state is stored as objects keyed by ID (e.g., `accounts: { [id]: Account }`), selectors frequently use `Object.values()` or `Object.keys()` to convert to arrays. This creates new array references on every selector evaluation, even when the underlying data hasn't changed, causing unnecessary re-renders and recomputations.

**Performance Impact:**

- **Memory allocation:** New arrays created on every selector evaluation
- **Garbage collection:** More frequent GC pauses from short-lived arrays
- **Re-renders:** Components re-render because array reference changes even if contents are identical
- **Cascading recomputations:** Downstream selectors that depend on these arrays recompute unnecessarily

```typescript
❌ WRONG: Creates new array on every call
export const getInternalAccounts = createSelector(
  (state: AccountsState) =>
    Object.values(state.metamask.internalAccounts.accounts), // New array every time!
  (accounts) => accounts, // Identity function doesn't help
);
```

**Solution 1: Store Arrays Alongside Objects**

```typescript
✅ CORRECT: Store both object and array in state
interface AccountsState {
  accounts: {
    byId: Record&lt;string, Account&gt;;
    allIds: string[]; // Maintained alongside byId
  };
}

// Selector uses pre-computed array
const selectAllAccounts = createSelector(
  (state) => state.accounts.allIds,
  (state) => state.accounts.byId,
  (allIds, byId) => allIds.map((id) => byId[id]),
);
```

**Solution 2: Proper Memoization**

```typescript
✅ CORRECT: Proper memoization with stable reference
// Base selector returns the object
const selectAccountsObject = (state: AccountsState) =>
  state.metamask.internalAccounts.accounts;

// Memoized conversion selector
export const getInternalAccounts = createSelector(
  selectAccountsObject,
  (accountsObject) => {
    // Only creates array when accountsObject reference changes
    return Object.values(accountsObject);
  },
);
```

**Solution 3: Normalize State Structure**

```typescript
✅ CORRECT: Normalized with indexes
{
  metamask: {
    nfts: {
      byId: { [nftId]: Nft },
      allIds: string[],
      byAccountId: { [accountId]: string[] }, // NFT IDs for account
      byChainId: { [chainId]: string[] }, // NFT IDs for chain
    }
  }
}

// Selector uses indexes instead of Object.values()
const selectNftsByAccount = createSelector(
  (state, accountId) => state.metamask.nfts.byAccountId[accountId] ?? [],
  (state) => state.metamask.nfts.byId,
  (nftIds, nftsById) => nftIds.map((id) => nftsById[id]),
);
```

### ❌ Anti-Pattern: Filtering/Searching Through Nested Objects

**Problem:** Selectors use `Object.values().find()` or `Object.values().filter()` to search through nested objects, which is O(n) and creates temporary arrays.

```typescript
❌ WRONG: Linear search through object values
export const getInternalAccountByAddress = createSelector(
  (state) => state.metamask.internalAccounts.accounts,
  (_, address) => address,
  (accounts, address) => {
    return Object.values(accounts).find((account) =>
      isEqualCaseInsensitive(account.address, address),
    );
  },
);
```

```typescript
✅ CORRECT: Maintain lookup index
// State includes address-to-ID mapping
interface AccountsState {
  accounts: {
    byId: Record&lt;string, Account&gt;;
    byAddress: Record&lt;string, string&gt;; // address -> accountId
  };
}

const selectAccountByAddress = createSelector(
  (state, address) => state.metamask.internalAccounts.accounts.byAddress[address.toLowerCase()],
  (state) => state.metamask.internalAccounts.accounts.byId,
  (accountId, accountsById) => accountId ? accountsById[accountId] : undefined,
);
```

---

---

## React Compiler Considerations

> **Tracking:** [Epic #6549: React Compiler Adoption](https://github.com/MetaMask/MetaMask-planning/issues/6549)

**Note:** This codebase uses React Compiler, a build-time tool that automatically optimizes React applications by memoizing components and hooks. React Compiler understands the [Rules of React](https://react.dev/reference/rules) and works with existing JavaScript/TypeScript code without requiring rewrites.

Reference: [React Compiler Introduction](https://github.com/reactwg/react-compiler/discussions/5)

### What React Compiler Does

React Compiler automatically applies memoization to improve update performance (re-renders). It focuses on two main use cases:

1. **Skipping cascading re-rendering of components** - Fine-grained reactivity where only changed parts re-render
2. **Skipping expensive calculations** - Memoizing expensive computations within components and hooks

### React Compiler Assumptions

React Compiler assumes your code:

- ✅ Is valid, semantic JavaScript
- ✅ Tests nullable/optional values before accessing (e.g., enable `strictNullChecks` in TypeScript)
- ✅ Follows the [Rules of React](https://react.dev/reference/rules)

React Compiler can verify many Rules of React statically and will **skip compilation** when it detects errors. Install [eslint-plugin-react-compiler](https://www.npmjs.com/package/eslint-plugin-react-compiler) to see compilation errors.

### React Compiler Limitations

#### Single-File Compilation

**React Compiler operates on a single file at a time** - it only uses information within that file to perform optimizations. This means:

- ✅ Works well for most React code (React's programming model uses plain JavaScript values)
- ❌ Cannot see across file boundaries
- ❌ Cannot use TypeScript/Flow type information (has its own internal type system)
- ❌ Cannot optimize based on information from other files

**Impact:** Code that depends on values from other files may not be optimized as effectively.

#### Effects and Dependency Memoization (Open Research Area)

**Effects memoization is still an open area of research.** React Compiler can sometimes memoize differently from manual memoization, which can cause issues with effects that rely on dependencies not changing to prevent infinite loops.

**Recommendation:**

- ✅ **Keep existing `useMemo()` and `useCallback()` calls** - Especially for effect dependencies to ensure behavior doesn't change
- ✅ **Write new code without `useMemo`/`useCallback`** - Let React Compiler handle it automatically
- ⚠️ React Compiler will statically validate that auto-memoization matches existing manual memoization
- ⚠️ If it can't prove they're the same, the component/hook is safely skipped over

### When Manual Memoization is Still Required

Due to React Compiler's **single-file compilation** limitation and inability to see across file boundaries, manual memoization is required for:

1. **Cross-File Dependencies** - Functions imported from other files
2. **Redux Selectors** - External state management
3. **External Hooks/Libraries** - Values from node_modules
4. **Effect Dependencies** - Keep existing useMemo/useCallback
5. **Refs and DOM Values** - Mutable values React can't track
6. **Third-Party Components** - Callbacks passed to external components

### Decision Tree: Do You Need Manual Memoization?

```text
Is the computation/value:
├─ From another file (imported function/hook)? → ✅ Manual memoization required
├─ From Redux selectors? → ✅ Manual memoization required
├─ From external library (node_modules)? → ✅ Manual memoization required
├─ Used as useEffect dependency? → ✅ Keep existing useMemo/useCallback
├─ Depends on refs or DOM values? → ✅ Manual memoization required
├─ Combines multiple cross-file dependencies? → ✅ Manual memoization required
└─ Simple props/state within same file? → ❌ React Compiler handles it

Is the callback:
├─ Used as useEffect dependency? → ✅ Keep existing useCallback
├─ Passed to external component/library? → ✅ Manual useCallback required
├─ Depends on imported functions/hooks? → ✅ Manual useCallback required
└─ Simple prop handler within file? → ❌ React Compiler handles it
```

### Summary: React Compiler Capabilities and Limitations

**React Compiler CAN optimize:**

1. ✅ Components and hooks within the same file
2. ✅ Expensive calculations within components/hooks
3. ✅ Fine-grained reactivity (preventing cascading re-renders)
4. ✅ Inline objects/functions with React-controlled dependencies
5. ✅ Derived state from props/state within the file
6. ✅ Simple conditional memoization based on props/state

**React Compiler CANNOT optimize:**

1. ❌ Code across file boundaries (single-file compilation)
2. ❌ Functions/hooks imported from other files
3. ❌ Redux selectors and external state management
4. ❌ Components from external libraries (node_modules)
5. ❌ Computations dependent on refs or DOM values
6. ❌ TypeScript/Flow type information (uses own type system)
7. ⚠️ Effect dependencies (keep existing useMemo/useCallback - open research area)

**Key Limitations:**

- **Single-file compilation** - Cannot see across files
- **No type information** - Doesn't use TypeScript/Flow types
- **Effects memoization** - Still an open research area

**Best Practices:**

- ✅ Write new code without `useMemo`/`useCallback` - let React Compiler handle it
- ✅ Keep existing `useMemo`/`useCallback` for effect dependencies
- ✅ Use manual memoization for cross-file dependencies
- ✅ Install [eslint-plugin-react-compiler](https://www.npmjs.com/package/eslint-plugin-react-compiler) to catch compilation errors

**Rule of thumb:** If it's within the same file and uses props/state, React Compiler handles it. If it crosses file boundaries (imports, Redux, external libraries), use manual memoization.

---
