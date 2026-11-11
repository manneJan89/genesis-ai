# Performance: client profile, React
- Stable references for props and hook dependencies (`useMemo`, `useCallback`
  where a child or effect depends on identity); never an object literal in a
  dependency array.
- Stable `key`s from data, not array index, for lists that change.
