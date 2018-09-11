# SCSS and Autoprefixer setup with Parcel

I recently built a small project using just HTML, SCSS and JavaScript and instead of using one of my old gulpfiles or writing an npm script for node-sass, I used Parcel instead.

## Parcel is Webpack without the configuration

Like Webpack, Parcel is a bundler for your web project but it's already preconfigured just the way I like it.

When I'm starting a simple SCSS website, I just want a few things:

- Auto SCSS to CSS compile
- Auto reloading or Hot module replacement (whatever I can get)
- Autoprefixer

Even for something simple like a SCSS setup, Parcel made a huge difference for me in reducing my setup time. My hope is that you can skip Gulp, Grunt or Webpack and opt for using Parcel instead because it's just better than what I would write on my own.

## Installation and HTML File setup

Create a new project called `parcel-scss`:

```bash
mkdir parcel-scss
cd parcel-scss
npm init -y
```

Then install `parcel-bundler`:

```bash
npm i parcel-bundler -S
```

Then create your `index.html` file and a `styles.scss` file.
When we write the html file, note that we will create a `<link>` element that points to the `styles.scss` file instead of a css file.

```html
<html lang="en">

<head>
  <title>SCSS with Parcel</title>
  <link rel="stylesheet" href="styles.scss">
</head>

<body>
  <div>
    <h1>SCSS with Parcel</h1>
  </div>
</body>

</html>
```

Parcel looks at this index.html file for the inputs it needs to bundle. So far, we're giving it one asset called `styles.scss` that should be compiled to a CSS file. What I love about this is that your `index.html` file is already designed to be written with tags pointing to external CSS and JS files and parcel utilizes that to know about these assets to bundle. Rather than writing explicit entry points and outputs, the good old `index.html` file is totally fine.

Let's put some SCSS into `styles.scss`.

```scss
/**
* style.scss
*/

$white-clouds: #ecf0f1;
$blue-midnight: #2c3e50;

@mixin reset {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

* {
  @include reset;
}

h1 {
  background-color: $blue-midnight;
  color: $white-clouds;
}

div {
  display: grid;
}
```

## Scripts for Parcel

In `package.json` is where we can write our `build` and `dev` scripts.
Look at the `"scripts"` part of the file, yours should look like this:

```json
{
  "name": "parcel-scss",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "parcel index.html",
    "build": "parcel build index.html",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "parcel-bundler": "^1.9.7"
  }
}
```

- `npm run dev`: is your dev script. Run this whenever you want to work on your project and it will spin up a local dev server at http://localhost:1234
- `npm run build` is your build script. This will build your app for production.

Try running the dev script and watch your terminal:

```
npm run dev
```

![terminal](https://user-images.githubusercontent.com/4185382/45386049-010d1c80-b5d8-11e8-9536-05f24cc7405c.png)

Isn't that cool? We told parcel to use our index.html file and, naturally, it found the scss file. So now it's installing `node-sass` so it can compile it.

Take a look at your `package.json` file, you should now see that `node-sass` is installed for you in `devDependencies`:

```json
"devDependencies": {
  "node-sass": "^4.9.3"
}
```

Magical.

## Autoprefixer

If you look at the newly created `dist` foldder in your project root, you'll notice that we ain't got no prefixes in our css file.

Let's fix that.

Let's refer to the Parcel docs for this: https://parceljs.org/transforms.html#postcss

All we gotta do is install autoprefixer and setup the postcss config file aka `.postcssrc`:

```bash
npm i autoprefixer
```

```
{
  "plugins": {
    "autoprefixer": {
      "grid": true
    }
  }
}
```

My guess is that `"grid": true` probably is a special option that enables you to use css grid with vendor prefixes but 🤷‍♂️.

Lastly, we need a config file for the browsers we want to be compatible with aka `.browserslistrc`:

```
> 1%
last 2 versions
```

I feel like we should be able to move this list into `.postcssrc`...but again 🤷‍♂️

Restart your terminal and run the dev script again:

```
npm run dev
```

Now when you look back at your css file in dist, guess what, now you got prefixes.

```css
* {
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

h1 {
  background-color: #2c3e50;
  color: #ecf0f1;
}

div {
  display: -ms-grid;
  display: grid;
}
```
