## chapter 2

`clsx` help us toggle class name, like this

```tsx
<span
  className={clsx(
    'inline-flex items-center rounded-full px-2 py-1 text-sm',
    {
      'bg-gray-100 text-gray-500': status === 'pending',
      'bg-green-500 text-white': status === 'paid',
    },
  )}
>
```

## chapter 3

key: optimize fonts and image

**fonts**

custom fonts can affect performance, and may cause layout offset

solution: using `next/font` module, like this

```ts
// /app/ui/fonts.ts
import { Inter } from 'next/font/google';

export const inter = Inter({ subsets: ['latin'] });
```

It is then added to the component's className, like `<p className={`$(inter.className)`}/>`

**image**

- Preventing layout may shift automatically when images are loading.
- Resizing images to avoid shipping large images to devices with a smaller viewport.
- Lazy loading images

using `next/image` module, like this

```tsx
<Image
    src="/hero-desktop.png"
    width={1000}
    height={760}
    className="hidden md:block"
    alt="Screenshots of the dashboard project showing desktop version"
/>
<Image
    src="/hero-mobile.png"
    width={560}
    height={620}
    className="block md:hidden"
    alt="Screenshot of the dashboard project showing mobile version"
/>
```

it will display different image on mobile and desktop devices

## chapter 4

key: routes

`next.js` uses file-system routing

we can create separate UIs for each route using `layout.tsx` and `page.tsx` files.

`/app/dashboard/page.tsx` is associated with the `/dashboard` path.

`layout.tsx` file can create UI that is shared between multiple pages. Only the page components update while the layout won't re-render on navigation.

RootLayout in `/app/layout.tsx` is necessary. Any UI that add to the RootLayout will be shared across all pages in application. and can add metadata

## chapter 5

key: optimize navigation

using `<Link/>` component from `next/link` module to replace `<a/>` tag. Page will not do a full refresh, because Next.js automatically code splits application by rote segment.

A hook called `usePathname()` from `next/navigation` can get the url (must add React's `'using client'` directive to the top of the file).

## chapter 6

key: set up a PostgreSQL database using `@vercel/postgres`.

## chapter 7

key: fetch data, solve request waterfalls

method: API layer or Database queries

**API layer**:

**Database queries**:

ORM, SQL

## chapter 8

key: static rendering, dynamic rendering

for static rendering, user should refresh page to refetch data.

dynamic rendering provide real specific data

add `noStore()` in `fetchRevenue()` to prevent the response from being cached.

if `fetchInvoiceById` function does't add `unstable_noStore()`, will cause problems with the edit operation in Chapter 12 (the data on the edited page will remain unchanged after modification).

## chapter 9

key: streaming, loading skeletons, [route groups][route-groups], fallback component

streaming a whole page with `loading.tsx`

fix that loading skeleton apply to children pages by creating `(overview)` route groups, the route groups is a folder called `(xxx)`

steaming a component: wrap `<Suspense>...</Suspense>` around component, if don't pass fallback component, it will cause layout offset, like this:

```tsx
<Suspense fallback={<RevenueChartSkeleton />}>
  <RevenueChart />
</Suspense>
```

## chapter 10

skip

## chapter 11

key: search, pagination, debounce

Next.js client hooks thar use to implement the search functionality:

- `useSearchParams`: access the parameters of the current URL. For example, the URL is `/dashboard/invoices?page=1&query=pending`, than the parameters is `{page: '1', query: 'pending'}`.
- `usePathname`: access the current URL's pathname. For example, the URL is `/dashboard/invoices`, usePathname would return `'/dashboard/invoices'`
- `useRouter`: change the current URL(remind: it's from `next/navigation` module)

The URL is updated without reloading the page, because of [Next.js's client-side navigation][Next.js's_client-side_navigation]

implement debouncing by `useDebouncedCallback` from `use-debounce` module(`npm i use-debounce`), like this

```tsx
const handleSearch = useDebouncedCallback((term: string) => {
  const params = new URLSearchParams(searchParams);
  if (term) {
    params.set('query', term);
  } else {
    params.delete('query');
  }
  replace(`${pathname}?${params.toString()}`);
}, 500);
```

## chapter 12

key: form data, React Server Actions, type validation, revalidate the client cache, dynamic route segments

think about what happens when JavaScript is disabled or fails to load on the client side(prompt: client action and server action).

one benefit of using a Server Actions is [Progressive Enhancement][Progressive_Enhancement].

type validation and coercion(casts): a library called [Zod][Zod], like this

```tsx
'use server';

import { z } from 'zod';

const FormSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  amount: z.coerce.number(),
  status: z.enum(['pending', 'paid']),
  date: z.string(),
});

const CreateInvoice = FormSchema.omit({ id: true, date: true });

// ...
export async function createInvoice(formData: FormData) {
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  });
}
```

Revalidate and redirect: Next.js has a [Client-side Router Cache][Client-side_Router_Cache].

Dynamic route: `[folderName]` folder. the route name will pass as `{params}` prop to `layout`, `page`, `route` and `generateMetadata` function.

```tsx
// app/blog/[slug]/page.js
export default function Page({ params }) {
  return <div>My Post: {params.slug}</div>;
}
```

[pass id to the Server Action using JS bind][pass_id_to_the_Server_Action_using_JS_bind]: in order not to put the id in the form.

## chapter 13

key: handing errors

`error.tsx`, `not-found.tsx`, `notFound()`

## chapter 14

key: accessibility, form validation

`useFormState` hook


[route-groups]: https://nextjs.org/docs/app/building-your-application/routing/route-groups
[Next.js's_client-side_navigation]: https://nextjs.org/learn/dashboard-app/navigating-between-pages
[Progressive_Enhancement]: https://en.wikipedia.org/wiki/Progressive_enhancement
[Zod]: https://zod.dev/
[Client-side_Router_Cache]: https://nextjs.org/docs/app/building-your-application/caching#router-cache
[pass_id_to_the_Server_Action_using_JS_bind]: https://nextjs.org/learn/dashboard-app/mutating-data#4-pass-the-id-to-the-server-action
