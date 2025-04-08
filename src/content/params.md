---
 title: SearchParams not functioning correctly in Next.js 15
 date: 2024-10-22
 author: belirofon
 description: I found the fix, the generated types now dictate that params/searchParams is a promise so the fix is fairly simple.
---

I found the fix, the generated types now dictate that params/searchParams is a promise so the fix is fairly simple.

```typescript
// Generated types by Next.JS
export interface LayoutProps {
  children?: React.ReactNode

  params?: Promise<SegmentParams>
}

The fix should look like

// Your page/layout/template file
export default async function Layout(props: { children: ReactNode; params: Promise<{ id: string }> }) {
  const { children, params } = props;

  const { id } = await params;
  
  // ... do whatever
```

- [source](https://github.com/vercel/next.js/issues/71603)
- [more info](https://nextjs.org/blog/next-15)
