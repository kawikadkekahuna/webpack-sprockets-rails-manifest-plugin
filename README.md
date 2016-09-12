Webpack Sprockets Rails Manifest Plugin
=======================================

A webpack plugin that allows [`sprockets-rails`](https://github.com/rails/sprockets-rails) 
to find files from a webpack build without loading them via the asset pipeline.

Install
-------

```bash
npm install webpack-sprockets-rails-manifest-plugin --save
```

Usage
-----

```ruby
# config/initializers/assets.rb
# Avoid creating the manifest file in public/assets as it can be downloaded by anyone
Rails.configuration.assets.manifest = Rails.root.join("config", "sprockets-manifest.json")
```

```erb
<% # app/views/layouts/application.html.erb %>
<% # Use rails helpers in the same way you normally would with sprockets %>
<!DOCTYPE html>
<html lang="en">
  <head></head>
  <body>
    <%= javascript_include_tag "webpack-application-bundle" %>
  </body>
</html>
```

```javascript
// frontend/bundles/webpack-application.js
alert("Howdy, Webpack is working!");
```

```javascript
// frontend/webpack.config.js
var WebpackSprocketsRailsManifestPlugin = require("webpack-sprockets-rails-manifest-plugin");

var config = {
  context: __dirname,

  entry: {
    "webpack-application-bundle": "./bundles/webpack-application"
  },

  output: {
    path: "../public/assets",
    filename: "[name]-[chunkhash].js"
  },

  // Other config ...

  plugins: [
    new WebpackSprocketsRailsManifestPlugin(
      // [optional] this is the default location relative to the output directory 
      // that the assets will be built. e.g `public/assets`
      manifestFile: "../../config/sprockets-manifest.json"
    )
  ]
};

module.exports = config;
```

Heroku
------

Follow the [instructions](https://devcenter.heroku.com/articles/nodejs-support) 
to install the ruby and node multi-buildpack. 

Create a `package.json` file in the root of the rails project.

```json
{
  "name": "MyProject",
  "version": "1.0.0",
  "description": "A description of MyProject",
  "main": "index.js",
  "engines": {
    "node": "5.12.0",
    "npm": "3.10.6"
  },
  "cacheDirectories": [
    "frontend/node_modules"
  ],
  "scripts": {
    "preinstall": "cd frontend && npm install",
    "postinstall": "cd frontend && npm run build"
  }
}
```

How Does It Work?
-----------------

`sprockets-rails` provides two strategies to map logical paths (`webpack-application.js`) 
to file system paths (`webpack-application-8f88619b6ef3a358a7ad.js`).

#### Environment

Searches the default sprockets directory for a file type manifest. By default 
this is only enabled in the `dev` and `test` environments e.g.

```javascript
// app/assets/javascripts/application.js
//= require_self
alert("Hi, I'm a Javascript manifest in the default Sprockets Asset Pipeline");
```

#### Manifest

A json file which stores a mapping between a compiled asset and the original 
logical path, along with meta information such as size, creation time and 
integrity hash. `webpack-sprockets-rails-manifest-plugin` uses this manifest file
to make `sprockets-rails` aware of the files and meta information in the 
`webpack` build. Since Rails 3.x `sprockets-rails` writes it's manifest file to 
a dynamic location using a randomly generated filename (e.g. `public/assets/.sprockets-manifest-b4b2e3829c91f2f867c051a855232bed.json`).
To get `sprockets-rails` and `webpack` talking to each other we need to 
configure `sprockets-rails` to write it's manifest file to a static location 
that `webpack` knows about.

If `Rails.configuration.assets.compile = true` the asset pipeline will check the 
manifest first, if an entry can't be found, it falls back and checks the file 
system through the environment strategy.


TODO
----
* [ ] - sample project
* [ ] - gradual migration documentation
* [ ] - tests
* [ ] - support webpack-dev-server
* [ ] - [`script` integrity](https://w3c.github.io/webappsec-subresource-integrity)

License
-------

[MIT](https://opensource.org/licenses/MIT)
