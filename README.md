# vue-hmr-repro

HMR only works if the component has an active instance. A regression was [introduced in 3.2.11](https://github.com/vuejs/vue-next/commit/aa8908a8543c5151a2cc06ed4d8fab3a1461692a#diff-ccebe74771d12151844d4d2de4cf16c6f7fb7ed6584d30964dae54a23454a942L117). [`3.2.10` behaves as expected](https://github.com/ElMassimo/vite-vue-router-hmr-repro/tree/3.2.10).

Most users experience this issue when using `vue-router`, as no full reload occurs, and is not unusual to navigate between pages.

It can happen in different scenarios, for example, a component that is conditionally displayed. If the user modifies the component file when the condition is false, and then the condition becomes true, the rendered component is outdated and won't reflect current changes until a page reload or another HMR update is sent.

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
