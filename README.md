# Tailwind / Svelte Demo integration

# Issues
If you see this issue, you're likely using svelte 3.3.0

See https://github.com/sveltejs/svelte/issues/5722#issuecomment-733895712

This is certainly not the exact right fix, but adding this in the Svelte `compiler.js` at around line 28636, just before the call to `get_replacement` allows everything to build without errors:

```js
if (processed.map?._mappings) {
    processed.map = processed.map.toJSON();
}
```


`
[!] (plugin svelte) TypeError: Cannot read property 'length' of undefined
src\App.svelte
TypeError: Cannot read property 'length' of undefined
    at sourcemap_add_offset (C:\devlocal\svelte-tailwind\node_modules\svelte\src\compiler\utils\string_with_sourcemap.ts:18:19)
    at get_replacement (C:\devlocal\svelte-tailwind\node_modules\svelte\src\compiler\preprocess\index.ts:112:3)
    at C:\devlocal\svelte-tailwind\node_modules\svelte\src\compiler\preprocess\index.ts:194:12
    at async Promise.all (index 0)
    at replace_async (C:\devlocal\svelte-tailwind\node_modules\svelte\src\compiler\preprocess\index.ts:71:48)
    at preprocess_tag_content (C:\devlocal\svelte-tailwind\node_modules\svelte\src\compiler\preprocess\index.ts:171:15)
    at preprocess (C:\devlocal\svelte-tailwind\node_modules\svelte\src\compiler\preprocess\index.ts:206:3)
    at ModuleLoader.addModuleSource (C:\devlocal\svelte-tailwind\node_modules\rollup\dist\shared\rollup.js:18312:30)
    at ModuleLoader.fetchModule (C:\devlocal\svelte-tailwind\node_modules\rollup\dist\shared\rollup.js:18368:9)
    at async Promise.all (index 0)
`


Hi! This is a quick repo demonstration to how to quickly and painlessly integrate Tailwind into the Svelte pipeline, no additional scripts required! The main concept is that you'll need to leverage the postcss plugins that Tailwind uses directly, instead of relying on the Tailwind CLI. 

Then, use a `global` style block to import in Tailwind postcss plugins (check out src/App.svelte) to bring in Tailwind in!

# Steps

**Add Dependencies**

`yarn add -D tailwindcss autoprefixer svelte-preprocess`

**Setup rollup config**

Check out the `rollup.config.js` for the full setup, but it's just plucked from the tailwindcss Getting Started walkthrough!

**1. Add the following blocks into your rollup.config**
```
const preprocess = sveltePreprocess({
  sourceMap: !production,
  postcss: {
    plugins: [
      require("tailwindcss"),
      require("autoprefixer"),
    ],
  },
});
```

**2. Add `preprocess` into the `svelte` rollup plugin**


```
plugins: [
  svelte({
    ...
    preprocess
  })
]
```

**3. Add a \<style global> block that references Tailwind**

In this example, I've added this block to the `src/App.svelte` component.  

## **Important**

In an actual app, put this block in a file that will not change. Tailwind has relatively fast incremental builds, but the initial build goes through the entire 3mb+ worth of Tailwind CSS and is quite slow.


```css
<style global>
  /* purgecss start ignore */
  @tailwind base;
  @tailwind components;
  /* purgecss end ignore */
  @tailwind utilities;
</style>
```

**4. Done!**
