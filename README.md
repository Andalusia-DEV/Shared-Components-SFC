# Single File Components (SFC)
My research's result of how we can create Vue/Nuxt shared components.


## Step 1: Create NPM account and add organization to it

1. creating a normal account on [NPM](https://www.npmjs.com/signup).

2. Create a new Organization as **Unlimited public packages**.

## Step 2: Intialize new package (Component)

1. Inside CMD write `npm init --scope=Organization_name` and follow the initialization steps.

2. Install [rollup](https://rollupjs.org/guide/en/) module bundler with `npm install --global rollup`. You need to install this package once as it's global package. The **package structure** should be like this:

```
package.json
build/
   rollup.config.js
src/
   wrapper.js
   my-component.vue
dist/
README.md
```

3. Install the following packages as **Developer Dependencies**

```
npm i -D @rollup/plugin-buble
npm i -D @rollup/plugin-commonjs
npm i -D rollup-plugin-vue
npm i -D vue-template-compiler
```

4. Set `rollup.config.js` configures (Needs more research)

```js
import commonjs from '@rollup/plugin-commonjs'; // Convert CommonJS modules to ES6

import vue from 'rollup-plugin-vue'; // Handle .vue SFC files
import buble from '@rollup/plugin-buble'; // Transpile/polyfill with reasonable browser support

export default {
    input: 'src/wrapper.js', // Path relative to package.json
    output: {
        name: 'MyComponent',
        exports: 'named',
    },
    plugins: [
        commonjs(),
        vue({
            css: true, // Dynamically inject css as a <style> tag
            compileTemplate: true, // Explicitly convert template to render function
        }),
        buble(), // Transpile to ES5
    ],
};
```

5. Set `wrapper.js` file (Needs more research)

```js
// Import vue component
import component from './my-component.vue';

// Declare install function executed by Vue.use()
export function install(Vue) {
	if (install.installed) return;
	
    install.installed = true;
	Vue.component('MyComponent', component);
}

// Create module definition for Vue.use()
const plugin = {
	install,
};

// Auto-install when vue is found (eg. in browser via <script> tag)
let GlobalVue = null;

if (typeof window !== 'undefined') {
	GlobalVue = window.Vue;
} else if (typeof global !== 'undefined') {
	GlobalVue = global.Vue;
}

if (GlobalVue) {
	GlobalVue.use(plugin);
}

// To allow use as module (npm/webpack/etc.) export component
export default component;
```

6. Create you Vue/Nuxt component, preferred without *CSS* so each project can have it's own. For example:

```js
<template>
  <section :class="`${blockName}`">
    <h1 :class="`${blockName}__headline`">HELLO, {{ name }}</h1>
    <div :class="`${blockName}__wrapper`" v-for="post in posts" :key="post.id">
      <h2 :class="`${blockName}__title`">{{ post.title }}</h2>

      <p :class="`${blockName}__description`">{{ post.description }}</p>
    </div>

    <client-only>
      <splide :options="options">
        <splide-slide v-for="post in posts" :key="post.id">
          <div class="doctor">
            <figure class="doctor__figure">
              <img
                :src="post.photo"
                :alt="post.image_alt == null ? 'Doctor Name' : post.image_alt"
                class="doctor__img"
              />

              <div class="socials">
                <a href="#" class="socials__link">
                  <i class="fab fa-facebook-square socials__icon"></i>
                </a>

                <a href="#" class="socials__link">
                  <i class="fab fa-twitter-square socials__icon"></i>
                </a>

                <a href="#" class="socials__link">
                  <i class="fab fa-instagram-square socials__icon"></i>
                </a>

                <a href="#" class="socials__link">
                  <i class="fab fa-linkedin socials__icon"></i>
                </a>

                <a href="#" class="socials__link">
                  <i class="fab fa-youtube-square socials__icon"></i>
                </a>
              </div>
            </figure>

            <div class="doctor__info">
              <h4 class="doctor__name">{{ post.title }}</h4>
              <h6 class="doctor__position">{{ post.position }}</h6>
              <p class="doctor__about">
                {{ post.small_description }}
              </p>
              <div
                class="nav-item nav-item--btn nav-item--normal nav-item--doctors m-0"
              >
                <nuxt-link
                  :to="localePath('/online-consultation')"
                  class="nav-link nav-link--btn nav-link--normal nav-link--doctors"
                >
                  {{ $t("reserveAppointment") }}
                </nuxt-link>
              </div>
            </div>
          </div>
        </splide-slide>
      </splide>
    </client-only>
  </section>
</template>

<script>
export default {
  name: "ComponentName",
  props: ["blockName", "name", "posts"],
  // SEO WORKS FINE TOO
  head() {
    return {
      title: "Budi Salah Component!",
      meta: [
        {
          hid: "description",
          name: `description`,
          content: `This is content meta for SEO!`,
        },
      ],
    };
  },
  data() {
    return {
      options: {
        rewind: true,
        perPage: 4,
        direction: this.$i18n.locale == "en" ? "ltr" : "rtl",
        arrows: false,
        gap: "40px",
        classes: {
          pagination: "splide__pagination doctors__pagination",
        },
        breakpoints: {
          992: {
            perPage: 2,
          },
          600: {
            perPage: 1,
            padding: {
              right: "5rem",
              left: "5rem",
            },
          },
          425: {
            perPage: 1,
            padding: {
              right: "0",
              left: "0",
            },
          },
        },
      },
    };
  },
  mounted: function () {
    this.$nextTick(function () {
      console.log("Console.log from this.$nextTick");
    });
  },
};
</script>
```

7. Inside your terminal write `npm run build` to build the package (Component).

8. Update `README.md` file to document your new changes and modifications. This step is **very very important**.

9. Update your package version for example `"version": "1.0.9"`. This step is also important and **you can't publish** your package (Component) without it.

10. Publish your component on NPM with `npm publish --access public` command. Now you ready to use this component across all of your projects.

## Step 3: Use your NPM component inside your project

1. Install the component like any other normal NPM package.

```
npm i @organization_name/component_name
```

2. Import the component to whatever page/component you want to use it inside.

```js
// pages/index.vue

<template>
    <ComponentName name="Budi Salah" :posts="postsFetch" blockName="hero" />
</template>

<script>
import ComponentName from '@organization_name/component_name';

export default {
  components: {
    ComponentName,
  },
    ...
},
</script>
```

## Notes

- This method makes it very easy to update and maintenance your component. You only need to code once and update the componant across your applications with `npm update @organization_name/component_name`.

- Developers can use any **version** of the package (Component), That's makes it very flexable.

- This method is SSR friendly.

- This method has been tested for the build version and no issues have been found so far.

## Resources

#### Single File Component (SFC)
- https://www.voorhoede.nl/en/blog/turning-vue-components-into-reusable-npm-packages/
- https://vuejs.org/v2/guide/single-file-components.html
- https://www.youtube.com/watch?v=iDCt8DtJr3s&ab_channel=VueMastery
- https://www.youtube.com/watch?v=xDDhovzgeT0&ab_channel=HasgeekTV
- https://vuejs.org/v2/guide/single-file-components.html)

#### Packaging Vue Components for NPM
- https://vuejs.org/v2/cookbook/packaging-sfc-for-npm.html)