# Using node-sass

> The team at `create-react-app` already has an [awesome walkthrough](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#adding-a-css-preprocessor-sass-less-etc) for using node-sass in a create-react-app project. Check it out!

## Tutorial

This setup will let you do a few things:

- Compile SCSS to CSS
- Watch for changes on any `*.scss` files in src/scss
- Automatically add vendor-prefixes to `*.css` files with `autoprefixer`. 

## Install

You can use `node-sass` or `node-sass-chokidar`. I'm going to use `node-sass-chokidar` because that's what the `create-react-app` people did `¯\_(ツ)_/¯`

```sh
npm init -y
yarn add node-sass-chokidar -S
# npm i node-sass-chokidar
```

Then install all this other stuff

```sh
yarn add autoprefixer chokidar-cli npm-run-all postcss-cli -S
# npm i autoprefixer chokidar-cli npm-run-all postcss-cli -S
```

## Scripts

```json
{
  "scripts": {
    "build-task:scss-compile": "node-sass-chokidar --source-map true src/scss/ -o dist/css",
    "build-task:autoprefixer": "postcss dist/css/*.css --use autoprefixer -d dist/css",
    "sass:build": "npm-run-all -p build-task:*",
    "sass:watch": "chokidar 'src/scss/**/*.scss' -c 'npm run sass:build'",
    "dev": "npm-run-all -p sass:*"
  },
}
```

> **On Windows**: Be sure to wrap your double-quotes with double-quotes AND escape those inner double-quotes! 
Here's a Windows friendly version of the watch script: 
> ```
> "watch": "chokidar \"src/scss/**/*.scss\" -c \"npm run sass:build\""
>```


When I work on projects, I run my dev environment using `npm run dev`; this will be the script we run to do all the SCSS things. 

For modularity, `autoprefixer` and `compile` get their own scripts. Both are called with `npm run sass:build`.

<!--TODO: Explain build-task:scss-compile-->

<!--TODO: Explain build-task:autoprefixer-->

<!--TODO: Explain sass:build w npm-run-all -p-->

<!--TODO: Explain sass:watch w chokidar -c-->

Finally, that brings us back to `npm run dev`, which runs all of our `sass:*` scripts in parallel. 

If you eventually have other scripts you need to run, you can throw them into the dev script.

```json
{
  "dev": "npm-run-all -p sass:* js:* html:*"
}
```

## Refactoring (optional)

<!--TODO: CLI to node.js files-->
