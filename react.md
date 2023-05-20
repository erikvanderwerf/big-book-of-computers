> React is a frontend JavaScript library for building user interfaces based on components.

&ndash; [Wikipedia](https://en.wikipedia.org/wiki/React_(software))

# NVM, NPM, NPX, and Node

Use [nvm](https://aur.archlinux.org/packages/nvm) to collect a version of npm, npx, and node.

```bash
nvm install --latest-npm
nvm list
nvm alias default ...
```

# [Create React App](https://create-react-app.dev/)

Set up a git repository for the project.
From the directory above the project root, run the `create-react-app` script in npx
using the project name/directory.

```bash
npx create-react-app <directory>
BROWSER=none npm start
```

Choose TypeScript and Webpack as prompted.
Remember to make a commit to save the repository progress as a working React app.

# [Webpack](https://webpack.js.org/)

Webpack is a build framework for frontend applications.
`create-react-app` bundles minimal support for Webpack by default, but in order to enable
TypeScript you need to prepare the project for it [explicitly](https://webpack.js.org/configuration/).

```bash
npx webpack init
```

You do not need to overwrite `README.md`.

# [Storybook](https://storybook.js.org/)

> Storybook is a frontend workshop for building UI components and pages in isolation

Initialize Storybook into the project through the Storybook npx init script.

```bash
npx storybook@latest init
npm run storybook dev --no-open -p 6006
```
