# SmartBeans Documentation

This repository contains the source code of the **SmartBeans** documentation site, built with [Docusaurus](https://docusaurus.io/).

The site is automatically deployed to **GitHub Pages** via **GitHub Actions** and is available at:  
[https://beanssmart.github.io/smartbeans-doc/](https://beanssmart.github.io/smartbeans-doc/)

---

## Getting Started

### Prerequisites
- [Node.js](https://nodejs.org/) (recommended: version 18 or newer)
- [Yarn](https://yarnpkg.com/) or [npm](https://www.npmjs.com/)

### Local Development

````bash
yarn start
# or
npm run start
````

The server will be available at http://localhost:3000 and supports hot-reload.

### Build for Production

````bash
yarn build
# or
npm run build
````
The static files will be generated in the `build` directory. You can test the production build locally with:

````bash
yarn serve
# or
npm run serve
````

### Deployment

Deployment is automated with GitHub Actions: on every push to the `master` branch, the workflow builds the site and
deploys it to GitHub Pages.

## Resources

* [Docusaurus Documentation](https://docusaurus.io/docs)
