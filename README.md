 # Webpacker + Jest

 1. Rails. I generated a new Rails app, skipping all of the things that I'm not likely to need.

        rails new rails-react --skip-action-mailer --skip-active-record --skip-action-cable --skip-javascript --skip-turbolinks

 2. Until Rails 5.1 ships, we need Webpacker.

        # Gemfile
        gem 'webpacker', git: 'https://github.com/rails/webpacker.git'

    Then in your shell

        $ bundle install

 3. Set up Webpack and React using Webpacker

        $ rails webpacker:install && rails webpacker:install:react

 4. Install Jest, and Jest related packages

        $ bin/yarn add jest babel-jest babel-polyfill react-test-renderer --dev

 5. Configure Jest to play nice with Webpacker. [This article][1] on configuring Jest for Webpack is very helpful. According to the docs "`<rootDir>` is a special token that gets replaced by Jest with the root of your project. Most of the time this will be the folder where your package.json is located". Crucially for us this is `/vendor` and not `/` as it would be in a conventional JavaScript project.

  ```javascript
  // vendor/package.json
  "jest": {
      // The name of this directive will soon change to "roots"
      // https://github.com/facebook/jest/issues/2600
      // This points Jest at Webpacker's default JS source directory
      "testPathDirs": ["<rootDir>/../app/javascript/packs"],
      // This ensures that node_modules are resolved properly
      // By default, Jest looks in "/" for this, not "vendor/"
      "moduleDirectories": ["<rootDir>/node_modules"],
      "moduleFileExtensions": ["js","jsx"]
  }
  ```

 6. Babel. This one caused me the most headache. Babel doesn't support dynamic configuration (although it is [currently being considered][2]). Additionally, the way Babel looks for configuration does not help us. "Babel will look for a .babelrc in the current directory of the file being transpiled. If one does not exist, it will travel up the directory tree until it finds either a .babelrc, or a package.json with a "babel": {} hash within."

  The way Babel looks for presets is also brittle. You'll note that [the babel-handbook documentation for creating presets][3] include the step "simply publish this to npm and you can use it like you would any preset". If you want Babel to actually find a preset in the conventional way, there is no alternative to using an npm package inside of the `node_modules` directory with a name starting with `babel-preset`.

  This means that we need to create a `.babelrc` file in the Rails root, and then use the full path to each plugin that we need, relative to that `.babelrc` file, rather than the short-hand that you'll be familiar with if you've used Babel in the past (e.g. `"presets": ["react"]` refers to `node_modules/babel-preset-react`). My `.babelrc`, following the Jest docs on [using Webpack 2](https://facebook.github.io/jest/docs/webpack.html#using-with-webpack-2) and [using babel](http://facebook.github.io/jest/docs/getting-started.html#using-babel) looks like this:

  ```javascript
  // .babelrc
  {
      "presets": [
        "./vendor/node_modules/babel-preset-react",
        "./vendor/node_modules/babel-preset-es2015"
      ],
      "env": {
        "test": {
          "plugins": [
            "./vendor/node_modules/babel-plugin-transform-es2015-modules-commonjs"
          ]
        }
      }
  }
  ```
  Remove the `options` key under the `babel-loader` configuration in `config/webpack/shared.js` so that Webpack uses this configuration file, too.

[1]: https://facebook.github.io/jest/docs/webpack.html
[2]: https://github.com/babel/babel/pull/4892
[3]: https://github.com/thejameskyle/babel-handbook/blob/master/translations/en/user-handbook.md#making-your-own-preset
