# chooseRoom

## Build Setup

```bash
# install dependencies
$ npm install

# serve with hot reload at localhost:3000
$ npm run dev

# build for production and launch server
$ npm run build
$ npm run start

# generate static project
$ npm run generate
```

For detailed explanation on how things work, check out the [documentation](https://nuxtjs.org).

## Special Directories

You can create the following extra directories, some of which have special behaviors. Only `pages` is required; you can delete them if you don't want to use their functionality.

### `assets`

The assets directory contains your uncompiled assets such as Stylus or Sass files, images, or fonts.

More information about the usage of this directory in [the documentation](https://nuxtjs.org/docs/2.x/directory-structure/assets).

### `components`

The components directory contains your Vue.js components. Components make up the different parts of your page and can be reused and imported into your pages, layouts and even other components.

More information about the usage of this directory in [the documentation](https://nuxtjs.org/docs/2.x/directory-structure/components).

### `layouts`

Layouts are a great help when you want to change the look and feel of your Nuxt app, whether you want to include a sidebar or have distinct layouts for mobile and desktop.

More information about the usage of this directory in [the documentation](https://nuxtjs.org/docs/2.x/directory-structure/layouts).


### `pages`

This directory contains your application views and routes. Nuxt will read all the `*.vue` files inside this directory and setup Vue Router automatically.

More information about the usage of this directory in [the documentation](https://nuxtjs.org/docs/2.x/get-started/routing).

### `plugins`

The plugins directory contains JavaScript plugins that you want to run before instantiating the root Vue.js Application. This is the place to add Vue plugins and to inject functions or constants. Every time you need to use `Vue.use()`, you should create a file in `plugins/` and add its path to plugins in `nuxt.config.js`.

More information about the usage of this directory in [the documentation](https://nuxtjs.org/docs/2.x/directory-structure/plugins).

### `static`

This directory contains your static files. Each file inside this directory is mapped to `/`.

Example: `/static/robots.txt` is mapped as `/robots.txt`.

More information about the usage of this directory in [the documentation](https://nuxtjs.org/docs/2.x/directory-structure/static).

### `store`

This directory contains your Vuex store files. Creating a file in this directory automatically activates Vuex.

More information about the usage of this directory in [the documentation](https://nuxtjs.org/docs/2.x/directory-structure/store).

### How to do weights

"Feature" is like an input variable. Features include Clarity, T20, room size, etc.
Every room should all have the same features. 

"Selection Weight" is associated with choice. When user make a choice, it indicates the preference of user on a certain value. This is pre-determined by us. Selection Weight is a number between 100 and -100.

"Question Weight" is associated with questions. It indicates the how much the user "values" this particular question, i.e. this particular value. Question Weight is a number between 100 and -100.

So how do we calculate the probability map? we use the following scheme:

Step 1: Initialize all room with probability 1;

Step 2: For every room:
            For every feature:
                Probability of Room += tanh(feature input value) * (Selection Weight / 100) * (Question Weight / 100);

The reason we use tanh function here, is to squash the input value into a scale of -1 to 1, so they are "normalized". This would help with reducing drastic changes in probability estimation.

After step 2, we have the probability for each room. Now we need to find a way to display this to the user in a useful way.

Step 3: Sort the probability map from high to low.

Step 4: Use sigmoid function to squash probability:
    For every probability:
        Probability = 1 / (1 + exp(Probability));

Using sigmoid could squash all values to 0 and 1. Also, we would anticipate all probabilities here to distribute around 0, so it's somewhat linear; the edge probability would be squashed harder here, so that it won't affect the readability of the results of next step too much.

Step 5: Normalization. Normalize the sorted probability map to make the largest probability 1.

Now, we could read from the sorted probability map.