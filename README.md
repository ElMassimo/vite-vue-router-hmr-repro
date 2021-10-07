# vue-router-hmr-repro

Reproduction of https://github.com/vitejs/vite/issues/131.

The bug occurs both for statically and dynamically imported components.

## Replication Steps

- Install packages
- Run `npm run dev`
- Visit `http://localhost:3000/about`
- Comment `<Welcome/>` in `src/views/Home.vue`
- Navigate to the _Home_ page
- The `<Welcome/>` component is still displayed

## Explanation 🐞

When the modified page component:

- is the current route, it has an active instance, so HMR works.
- is __not__ the current route, it has no active instances, so HMR does nothing.

As a result, when modifying a component that is not the current route, the HMR
update is sent as usual, but it doesn't update the module.

Then, when the route for the modified component is visited, the original module
is used, which doesn't include the changes since it was never updated by HMR.

### More Subtle for Dynamic Imports

To replicate the bug with lazy routes, the dynamic import must have been resolved.

For example, the bug will not occur in this sequence:

- Navigate to the _Home_ page
- Add an exclamation mark to the text in `src/views/About.vue`
- Navigate to the _About_ page using the nav link
- The modified text is displayed

But, if you proceed to:

- Navigate again to the _Home_ page using the nav link
- Add another exclamation mark to the text in `src/views/About.vue`
- Navigate to the _About_ page using the nav link
- The last change is not there
