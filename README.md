# Monorepo with Lerna

## Steps

### Init lerna on the repo

```
$ npx lerna init
```

### Modify `lerna.json` and `package.json` to use Workspaces feature

```
// package.json
{
  "name": "root",
  "private": true,
  "workspaces": ["packages/*"],
  "devDependencies": {
    "lerna": "^3.22.1"
  }
}
```

```
// lerna.json
{
  "packages": ["packages/*"],
  "npmClient": "yarn",
  "useWorkspaces": true,
  "version": "independent"
}
```

We’ll use independent versioning so we can properly enforce semver for each
package.

### Install Babel dependencies

Using `-w` instructs Yarn to install the given dependencies for the entire
workspace. These dependencies are usually shared between all packages.

```
$ yarn add --dev -W @babel/cli @babel/core @babel/preset-react @babel/preset-env babel-core@7.0.0-bridge.0 babel-loader babel-plugin-styled-components webpack
```

And create a `babelrc.json` file on the root

```
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": ["babel-plugin-styled-components"]
}
```

And add a script on `package.json` to execute Babel

```
"scripts": {
    "build": "lerna exec --parallel -- babel --root-mode upward src -d lib --ignore **/*.stories.js,**/*.spec.js"
}
```

lerna exec will take any command and run it over all of the different packages.
This command instructs Babel to run in parallel over every package, pulling from
the /src folder and compiling into the /lib folder.

Using `--root-mode upward` is the special sauce to using Yarn workspaces. This
tells Babel the `node_modules` are located in the root instead of nested inside
each of the individual packages. This prevents each package from having the same
`node_modules` and extracts them up to the root. We’ll be utilizing a similar
approach for testing later.

### React
