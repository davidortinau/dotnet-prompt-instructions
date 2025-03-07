## Context

A flatter visual tree is better since it reduces the number of measure and arrange passes needed to render the tree. For this reason, prefer a complex Grid layout over nesting multiple other layouts.

## Guidelines

Do not nest scrollable controls (ScrollView, CollectionView, ListView) within each other unless they scroll in different directions. It is OK to nest within a RefreshView or similar pull-to-refresh control.