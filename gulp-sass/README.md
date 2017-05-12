# gulp-sass

To this day, `gulp-sass` is still my preferred way for working with SCSS. My recommendation is to use this with static sites or server-rendered web apps. Anything where you're compiling a standalone CSS stylesheet.

## Tutorial

This is the minimum amount of stuff I need to work with SCSS. I would also include `browser-sync` in here for live reloads but for now, let's just focus on what we have here.

### What we're making

By the end of this tutorial, we will have three `gulp` tasks that we can use.

- `gulp sass:build`
  - compiles SCSS to CSS (`gulp-sass`)
  - Uses `autoprefixer` to automatically add vendor prefixes to our CSS (`gulp-autoprefixer`)
  - creates a CSS sourcemap (**main.css.map**) used for debugging our stylesheets. Inspecting styles in the browser will give us references to the relevant SCSS file instead of the CSS file. (`gulp-sourcemaps`)
- `gulp sass:watch`
  - watches our SCSS files and runs `gulp sass:build` anytime we make changes.
- `gulp sass`
  - initiates both `gulp sass:build` and `gulp sass:watch` as a single task. **This is the main task you'll call directly**.

### File Structure

Let's assume you're working with the following:
- **src/scss**: 
  - All of our SCSS files go here
  - You can have as many nested files and folders in here as long as they're in **src/scss**.
- **dist/css**: 
  - Our gulp task will compile our SCSS files and write them here as CSS files.

```
├── dist
│   ├── css
│   │   ├── main.css
│   │   └── main.css.map
│   └── index.html
├── gulpfile.js
├── package.json
└── src
    └── scss
        ├── components
        │   └── _buttons.scss
        ├── globals
        │   ├── _colors.scss
        │   └── _reset.scss
        └── main.scss
```

### Getting Started

Let's get started by doing a couple things:
- Initialize a new **package.json** 
- Install the dependencies

```sh
npm init -y
yarn add gulp gulp-sass gulp-autoprefixer gulp-sourcemaps -S 
# npm i gulp gulp-sass gulp-autoprefixer gulp-sourcemaps -S
```

Create a **gulpfile.js** in the root of the project and `require` the dependencies at the top of the file.

```js
const gulp = require('gulp');
const sass = require('gulp-sass');
const autoprefixer = require('gulp-autoprefixer');
const sourcemaps = require('gulp-sourcemaps');
```

### Writing the `sass:build` task

Start by writing a gulp task called `sass:build` that returns a `gulp.src()` to our SCSS files. We feed the source files for our task using `gulp.src()`,using a `globby` pattern. 

```js
gulp.task('sass:build', () => {
  return gulp.src('src/scss/*.scss')
    .pipe(gulp.dest('dist/css'))
});
```

️️Here we're saying, look inside `src/scss` folder for any SCSS file (`*.scss`) and do something to it.

For me, I always have a `main.scss` file located here that `@imports` all the other SCSS partials for my app. The reason why I'm using the `*.scss` syntax is because I can rename `main.scss` to something else if I need to.

The destination is defined by `gulp.dest()` will be the last step of this task.
This will use `.pipe()` to take whatever is in `src/scss/*.scss` and streams it into `dist/css`. 

If we were to run this task now, this would simply move the `main.scss` file to `dist/css`.

Between `gulp.src` and `gulp.dest`, we can use `.pipe()` to stream the contents of **main.scss` into our sourcemaps, our sass compiler, and autoprefixer before we write it to our destination. Let's do that now.

```js
gulp.task('sass:build', () => {
  return gulp
    .src('src/scss/*.scss')
    .pipe(sourcemaps.init())
    .pipe(
      sass({
        outputStyle: 'expanded',
      }).on('error', sass.logError)
    )
    .pipe(autoprefixer({ browsers: ['last 2 versions'] }))
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest('dist/css'));
});
```

This is our full, completed `sass:build` gulp task. Let's break down what's happening:

If we want to use `sourcemaps`, the first thing we need to do is stream our src contents to `.pipe(sourcemaps.init())`.
Once we're done doing everything we need to do with our SCSS files, we'll stream the compiled src content to `.pipe(sourcemaps.write('./'))` (the `./` path will resolve to whatever path we set in `gulp.dest`).

Next, we stream our content to `.pipe(sass())`. 

You can see that two things are happening. 

First, we're passing in an object to `sass()`. This will configure certain options for your sass compiler; I'm setting `{ ouputStyle: 'expanded' }` so that my CSS will be indented in the way I like. Note that `gulp-sass` uses `node-sass` under the hood ([see list of options](https://github.com/sass/node-sass#includepaths)). 

Second thing is that `sass().on()` thingy. 

Whenever there's a compile error in our sass, it will log an error message to the console. 

Autoprefixer is pretty straight-forward. 
We stream our compiled SCSS to autoprefixer and make sure that it adds vendor prefixes that are compaitible the `last 2 versions` of all browsers.

### Writing the `sass:watch` task

Here it is!

```js
gulp.task('sass:watch', ['sass:build'], () => {
  gulp.watch('src/**/*.scss', ['sass:build']);
});
```

Let's break it down!

That `['sass:build']` thing is new, right? This means that when we run `gulp sass:watch`, it's going to run `gulp sass:build` first, wait for it to be done, then start running its own task.

The task itself is just using `gulp.watch`. You may have guessed it but this is going to watch our SCSS files for changes. 

We give `gulp.watch` a globby pattern, but this time the pattern looks like `src/**/*.scss`. This means, look in the `src` folder, and all subfolders, and target any files with the `.scss` extension.


### Writing the `sass` task

The last-ish task that we write is the one we're going to use in development for future projects. 

```js
gulp.task('sass', ['sass:watch', 'sass:build']);
```

All this does is run `sass:watch` and `sass:build` tasks as one process when we execute `gulp sass`. 

### Final Code

All together now! 🎉

```js
const gulp = require('gulp');
const sass = require('gulp-sass');
const autoprefixer = require('gulp-autoprefixer');
const sourcemaps = require('gulp-sourcemaps');

gulp.task('sass:build', () => {
  return gulp
    .src('src/scss/*.scss')
    .pipe(sourcemaps.init())
    .pipe(
      sass({
        outputStyle: 'expanded',
      }).on('error', sass.logError)
    )
    .pipe(autoprefixer({ browsers: ['last 2 versions'] }))
    .pipe(sourcemaps.write('./'))
    .pipe(gulp.dest('dist/css'));
});

gulp.task('sass:watch', ['sass:build'], () => {
  gulp.watch('src/**/*.scss', ['sass:build']);
});

gulp.task('sass', ['sass:watch', 'sass:build']);

gulp.task('default', () => {
  console.log('Use npm scripts');
});
```

> Bonus: I usually like to map my build tasks to `npm` scripts because I can do `npm run` to see what scripts are available to me. As a convention, I'll have a `npm run dev` script for my build/watch tasks. 

