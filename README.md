# Microfrontends for Smarthealth

Smarthealth is an online health platform at your finger tips at scale, requires a sclable frontend solution.

I am going to show how to create frontend at scale as miciroservices(microfrontends), using [Single-spa](https://single-spa.js.org/) framework with separate projects using different single page frameworks (Angular 8, Angularjs, Reactjs) in three easy steps.
This assumes you have the the shell app as in the repo, which acts like a gateway for other apps.
 
The following knowledge is a prerequisite, I struggled setting Single-spa, then realised that I was missing this foundational knowledge.
... Webpack, this article may be a good read, [weebpack](https://developer.okta.com/blog/2019/12/09/angular-webpack).
... [Angular cli build api](https://blog.nrwl.io/angular-cli-demystifying-the-workspace-7f59ffaab4cb).

## Step 1: Create the application.

Create navbar. 

... Create new application.

```ng new navbar```

## Step 2: Install and Configure single-spa-angular.

```npm install single-spa-angular --save```

Add

```navbar/src/main.single-spa.ts```
        
With the following content
        e
```
const lifecycles = singleSpaAngular({
bootstrapFunction: singleSpaProps => {
singleSpaPropsSubject.next(singleSpaProps);
return platformBrowserDynamic().bootstrapModule(AppModule);
},
template: '<navbar-root />',
Router,
NgZone: NgZone,
});

export const bootstrap = lifecycles.bootstrap;
export const mount = lifecycles.mount;

export const unmount = lifecycles.unmount;
```

Add folder

  ```src/single-spa```
         
Add file

```src/single-spa/asset-url.ts```
         

With the following
        
```
export function assetUrl(url: string): string {
// @ts-ignore
const publicPath = __webpack_public_path__;
const publicPathSuffix = publicPath.endsWith('/') ? '' : '/';
const urlPrefix = url.startsWith('/') ? '' : '/'

return `${publicPath}${publicPathSuffix}assets${urlPrefix}${url}`;
}
```

Add file 
        
```
src/single-spa/single-spa-props.ts
```

With the following
```        
export const singleSpaPropsSubject = new ReplaySubject<SingleSpaProps>(1)

// Add any custom single-spa props you have to this type def
// https://single-spa.js.org/docs/building-applications.html#custom-props
export type SingleSpaProps = AppProps & {
}
```

Update 

```/src/app/app.component.ts selector as below```

To

```selector: 'navbar-root'```

Update
src/index.html as:

```<navbar-root></navbar-root>```


## Step 3: Install and Configure custom-webpack.

```
npm install @angular-builders/custom-webpack --save-dev
```

d. Configure single-spa.
Install @angular-builders/custom-webpack.

```npm install -save-dev @angular-builders/custom-webpack```

Add the following file to the root folder

```extra-webpack.config.js```

With the following:
    
```
const singleSpaAngularWebpack = require('single-spa-angular/lib/webpack').default

    module.exports = (angularWebpackConfig, options) => {
    const singleSpaWebpackConfig = singleSpaAngularWebpack(angularWebpackConfig, options)

    // Feel free to modify this webpack config however you'd like to
    return singleSpaWebpackConfig
}
```

In angular.json replace the following:

```
"builder": "@angular-devkit/build-angular:browser",
```

With

```
"builder": "@angular-builders/custom-webpack:browser",
```
Replace `"scripts": []` with the following:
    
```
"scripts": [],
"customWebpackConfig": {
    "path": "./extra-webpack.config.js"
}
```

Replace

``` "builder": "@angular-devkit/build-angular:dev-server",```
    
With
``` "builder": "@angular-builders/custom-webpack:dev-server",```

In package.json scripts section replace:

```
"start": "ng serve",
"build": "ng build",
```

With
```
"start": "npm run serve:single-spa",
"build": "npm run build:single-spa",
```

And add the following 
      
```
"build:single-spa": "ng build --prod --deploy-url /dist/navbar --output-hashing none",
    "serve:single-spa": "ng serve --disable-host-check --port 4301 --deploy-url http://localhost:4301/ --live-reload false",
```


e. Add and Test in Shell.

Register navbar in the shell project index.html

```
singleSpa.registerApplication(
  'navbar',
  function () {
    return System.import('navbar');
  },
  function (location) {
    return true;
  }
)
```

Add app in systemjs section.
        
```
navbar": "http://localhost:4301/main.js",
```

In root of navbar run:
        
```npm start```

Navigate the shell app url, you should see the navbar.


___

You can repeat steps 1,2 3 for each app.
Please let me know if you find this useful on [@CleophasMashiri](https://twitter.com/CleophasMashiri) 