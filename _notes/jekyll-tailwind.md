---
layout: note
title: Start a new blank project with Jekyll and Tailwind
tags: ruby, jekyll, tailwind
permalink: jekyll-tailwind
---

Jekyll is a static site generator developed in Ruby. It's probably one of the oldest and easiest to use. It comes with lots of things that I don't need by default. I always spend too much time tweaking instead of building. One day I realized I could create new Jekyll projects without all the boilerplate and start building from the beginning.

I've been using Jekyll a lot lately and finally decided to write a step by step guide so I don't forget.

To begin you need to install the Jekyll gem on your system:

```bash
$ gem install jekyll
```

Next, create a new project with Jekyll passing the `--blank` option:

```bash
$ jekyll new sitename --blank
```

Since we've used the `--blank` option, our project does not have a `Gemfile`. Now is time to create one:

```bash
$ bundle init
```

Now add `jekyll` and `webrick` to our newly created `Gemfile`. [According to the docs](https://jekyllrb.com/docs/), you'll only need webrick if you use Ruby 3 or greater.

```bash
$ bundle add jekyll webrick
```

Jekyll is ready. To run the server just execute:

```bash
$ bundle exec jekyll serve
```

To see the result, type `https://localhost:400` on your browser.

Since we installed a bare-bones version of Jekyll, what we can see is just a simple text:

```text
You’re ready to go!

Start developing your Jekyll website.
```

## TailwindCSS

We're going to follow the [official Tailwind docs](https://tailwindcss.com/docs/installation/using-postcss) to install it with PostCSS.

Somebody already did some work for us creating a [Jekyll PostCSS plugin](https://github.com/mhanberg/jekyll-postcss). We're going to use it.

Do the same we did with Jekyll and Webrick:

```bash
$ bundle add jekyll-postcss
```

Next step consist on letting Jekyll know we've added a plugin. To do so, open `_config.yml` and add the next config lines:

```yaml
plugins:
  - jekyll-postcss

postcss:
  cache: false
```

Now is time to use npm and install some packages:

```bash
$ npm install -D postcss autoprefixer cssnano tailwindcss
```

Following the [plugin's docs](https://github.com/mhanberg/jekyll-postcss#usage), create a new `postcss.config.js` file with the following content:

```js
module.exports = {
  plugins: [
    require("tailwindcss"),
    require("autoprefixer"),
    ...(process.env.JEKYLL_ENV == "production"
      ? [require("cssnano")({ preset: "default" })]
      : [])
  ]
};
```

Note that I also required `tailwindcss` here.

Now is finally time to add Tailwind. Create a new file `tailwind.config.js` with the following content:

```js
module.exports = {
  content: [
    "./**/*.{md,html}"
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

The only difference with the code in the docs is in the `content` key. I specified the path and extensions that I want Tailwind to work on.

Just a few more changes to get everyting working.

Jekyll works with SASS and we don't want it here. Update the extension of `assets/css/main.scss` to `.css` and replace the content of the file with:

```css
---
---

@tailwind base;
@tailwind components;
@tailwind utilities;
```

Those two groups of dashes are part of the _YAML front matter_ and are necessary for Jekyll to work properly. The next 3 lines that start with `@tailwind` are the framework directives. They provide all the CSS classes and a few extra niceties.

Now it's time to run the server again and hope that everything went well. This time use the `--livereload` option. This way the browser will automatically refresh everytime you change anything in your code.

```bash
$ bundle exec jekyll serve --livereload  
```

You must see the same text as before but in a different style. This is part of what Tailwind does.

To be sure that everyting is working as expected, try changing the background color. Go to `_layouts/default.html` and add `class="bg-green-300"` to the `body` tag. As soon as you save your file the browser should refresh. Now your website has the same background color as mine!