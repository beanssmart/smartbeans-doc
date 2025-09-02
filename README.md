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

## License

This project is licensed under the **Apache License 2.0**.

You may obtain a copy of the License at:

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an
**"AS IS" BASIS**, without warranties or conditions of any kind, either express or implied.  
See the License for the specific language governing permissions and limitations under the License.

