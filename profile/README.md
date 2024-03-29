# Searchspring Managed Snap Repositories

https://github.com/searchspring-implementations

Repositories in this organization are supported by Searchspring and are configured to use a Github action to handle the testing, bundling and deploying of the Snap code.

These repositories can also be viewed via the [Snapp Explorer](https://searchspring.github.io/snapp-explorer/).

## Directory Structure
Directory structures may vary from project to project, but should generally follow the structure below. 

```
project
 ┣ 📂 .github
 ┃ ┗ 📂 workflows
 ┃   ┗ 📄 deploy.yml
 ┣ 📁 public
 ┣ 📂 src
 ┃ ┣ 📁 components
 ┃ ┣ 📁 scripts
 ┃ ┣ 📁 styles
 ┃ ┗ 📄 index.js
 ┣ 📂 tests
 ┃ ┣ 📂 cypress
 ┃ ┃ ┗ 📂 integration
 ┃ ┃   ┗ 📄 results.spec.js
 ┃ ┗ 📄 cypress.json
 ┣ 📄 .gitignore
 ┣ 📄 package-lock.json
 ┗ 📄 package.json
```

## Node.js
Snap development is recommended to be done with Node.js v20 (Iron LTS). The [Snap Github Action](https://github.com/searchspring/snap-action) Node.js runtime version is also v20.

It is recommended to install Node.js via [NVM](https://github.com/nvm-sh/nvm) to easily switch between various versions.

## Branching
The default branch for each repository is called `production`. The code in this branch is used to build the javascript bundle used on the live site. This branch has [branch protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches#require-status-checks-before-merging) enabled which will restrict all commits to this branch to be done in a  pull request that has a passing `Snap Action` check. For this reason, any code changes must be made to a separate branch, tested, and then merged back into `production` via a pull request. 

Commits directly to `production` are prohibited.

## Github Action
On commit, each branch runs a Github action (deploy.yml) that executes the following steps:
* install packages
* build snap bundle
* run e2e tests (on pushes to any branch)
* run lighthouse tests (on pull requests against any branch)
* sync recommendations templates (`production` branch only)
* upload to CDN with invalidation

The entire build process plus invalidation can take up to 5 minutes to process. Failure at any step in the process will halt the entire workflow - the bundle will not be uploaded to the CDN.

### Install Packages
Node packages are installed via the `npm ci` command. This command uses the `packages-lock.json` file for dependency resolution; it is important that this file be committed on all `package.json` changes.

### Build Snap Bundle
The build process utilizes [Webpack](https://webpack.js.org/) to create the Snap bundle file(s). The number of files will depend on the complexity of the codebase and the code splitting configuration.

### Sync Recommendations Templates
> ⚠️ This step occurs only on push actions triggered by the `production` branch.

Any recommendations templates found in the project will be synchronized to the SMC by using the `snapfu recs sync` command. This happens only on the `production` branch.

### E2E Tests
> ⚠️ This step occurs on push actions to any branch *except* for the `production` branch.

End-to-end tests are run using [Cypress](https://www.cypress.io/). Each site should at a minimum run core tests of the build - additional tests can be added as needed. The tests are located in the `/tests/cypress/integration` directory.

### Lighthouse Testing
> ⚠️ This step occurs for all pull request triggered actions.

Lighthouse testing is run using [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci). Lighthouse testing is to be used to try to determine if any performance, SEO or accessibility metrics have changed. The test HTML document can be found in `/tests/lighthouse/public/lighthouse.html`.

### Create Tag
> ⚠️ This step occurs only for push actions triggered by the `production` branch. Tags in the repository should never be removed or added manually. Additionally, the value of `searchspring.version` in `package.json` should never be modified manually.

A new Github tag will be pushed to the repository during this step. The new tag version is determined firstly by the value of `searchspring.version` in `package.json` and secondly by the existence of tags at that version. If a tag does not yet exist for `searchspring.version` it will be created (eg. `0.43.0`) - if one does exist, the tag version is *revision* incremented (eg. `0.43.0-1`).
### Upload to CDN
After all other steps have completed, the bundle build files (`/dist` directory) and public files (`/public` directory) are pushed up to the Searchspring CDN (content delivery network) and the appropriate paths are invalidated (to clear the edge caching).

## Bundle Files
In the URL examples below, replace `[your_site_id]` with the valid six-digit alphanumeric site id for your site.

### Production Bundle
The production bundle should be used on live site implementations, it is uploaded to this URL:  
`https://snapui.searchspring.io/[your_site_id]/bundle.js`  

### Branch Bundles
All other branch bundles are uploaded to the CDN at this URL (replace `[branch_name]` with the name of the git branch):  
`https://snapui.searchspring.io/[your_site_id]/[branch_name]/bundle.js`  

## Integration
The bundle file is typically placed in a global header file on the site inside of the `<head>` tag. The sooner that the bundle file is loaded - the sooner that Searchspring results will render on the page.
```html
<script src="https://snapui.searchspring.io/[your_site_id]/bundle.js"></script>
```

## Branch Overrides
To preview a branch build, simply add the branch name as a `branch` parameter to the current URL. 

For example, if you are on `https://www.sellingthings.com` and you wanted to test the `fix-bug` branch, you would navigate to `https://www.sellingthings.com?branch=fix-bug` after which you would see the branch preview popup showing the details of the preview.

![image](https://raw.githubusercontent.com/searchspring/snap/main/images/branch-override.png)

Tag versions will also be uploaded to the CDN alowing usage of these in branch overrides.

For example, if you are on `https://www.sellingthings.com` and you wanted to test the `0.43.0` tag, you would navigate to `https://www.sellingthings.com?branch=0.43.0` which will trigger the load of that build.

## Local Development
Typically a site repository would be cloned locally, modified on a new branch (based on `production`), pushed to Github and then merged back into the `production` branch using a pull request.

### Install Packages
The build and development tooling needs to be installed first. Install the packages with `npm install`.

### Development Server
Start up the webpack development server with the `npm run dev` script - this will spin up a server at https://localhost:3333. The development server has built in live reloading and style injection enabled by default.

### E2E Testing
If Cypress tests are failing in the `Snap Action` it can be helpful to run these tests locally to debug them. This can be done either headlessly via the `npm run test` script (this is what the `Snap Action` uses), or with the Cypress UI by using both the `npm run dev` alongside the `npm run cypress` scripts.

### Lighthouse Testing
If the Lighthouse test is failing in the `Snap Action` it can be helpful to run the test locally to debug. Most commonly this test will fail due to Javascript errors or incorrect or missing target selectors. If default selectors need to be modified or added, you can alter the `tests/lighthouse/public/lighthouse.html` file as needed.

The Lighthouse test utilizes the final production build of the project bundle - so prior to testing you *must* run `npm run build` to generate the build. Once the bundle has been built, you can then run `npm run lighthouse` to kick off the Lighthouse testing. This test runs headlessly by default, but will output a URL in the terminal that you can visit to see what it may be rendering or what errors are occurring.

### Analyze Bundle
Webpack is configured by default to split the code into "chunks" when dynamic imports are used. Sometimes it can be helpful to know which chunk contains certain components or code; this can easily be done by using Webpack's analyze tool. You can view this by running the `npm run analyze` script.

## Dev Mode
Development mode provides additional logging and prevents the caching of API responses. When developing locally, this will be automatically set. To set this mode on a live site, add a `dev` query parameter: `https://www.sellingthings.com?dev`. This will set a cookie that keeps the site in development mode. To remove development mode, set the query parameter equal to `0` or `false`: `https://www.sellingthings.com?dev=false`.

## Step by Step Examples
Generic code change: https://github.com/searchspring-implementations/.github/blob/production/CHANGES.md  
Rollback a published change on `production` by reverting a tag: https://github.com/searchspring-implementations/.github/blob/production/REVERT.md
