# Build Once Deploy Many React
This is a concise solution that enables how to "Build Once and Deploy Many" for React Apps - which means build the React application binary-only once, then migrate the same binary file from Development to Non-Production and then to Production by loading configurations at application runtime.

## The Scenario
FooStore is an online shop that operates in Australia, New Zealand and potentially more countries in the future. Currently, all automated testing (linting, integration, E2E, etc) in CI take more than 15 minutes. 

Due to frequent releases, they need a solution to build the application / binary only once and deploy it to multiple environments rather than spending 15 minutes in CI for each deployment (e.g. deploying to 4 environments would take an hour as needed to build code in CI for 4 times, as some critical settings change in every environment).

Thus, FooStore wants their React Web App to be able to load configurations at runtime based on different environments - e.g. [Dev, Test, Staging, Prod-AU, Prod-NZ, Prod-US, etc] plus a CI/CD pipeline upgrade to achieve the "Build Once and Deploy Many" functionality, which means they need to run CI for 15 minutes only once and then deploy to as many environments as they want.

 

## The Challenge
There are 2 challenges behind the solution.

1 - By default, in a standard structured React App folder, when you run yarn/npm build or react-scripts build, React minifies and uglifies all .ts / .js files except files in ~/public folder.

As you can see from the screenshot, only the files in ~/public folder stay in the original format, and the remaining .js files have complied to .chunk.js which is almost impossible to look up and replace environment values for the application to load at runtime.

2 - Next we need to figure out how to use a relative path for React App to know where config file is present. E.g:

In local (yarn start / npm run start), it would know the config file is at ~/home/myreactApp/someDir/config.js

When people access via the browser, it would know the config file is at webserver://~/dist/someDir/config.js

## The Solution
The aim of this CI/CD pipeline design is not only to simplify the deployment process, by making it repeatable and easier to debug with automatic code auditing and testing in place but also to replace the config file based on the environment during the CD process to achieve Build Once and Deploy Many.

### Pipeline Design

For now, we only focus on  CI / Build step and CD / Replace config.js steps as they are the key to achieving Build Once and Deploy Many with a React application. To elaborate, the below steps are needed:

### 1. Prepare a config.js file
👉 Create a config.js file in the ~/public folder

👉 Create config.nonprod.js, config.prod.js files in the ~/public/env folder 

Explanation:

As mentioned in the challenge section, during the code build time, React minifies and uglifies all .ts / .js files except files in ~/public folder.

Hence, the ~/public folder is where we should create our config.js file, as well as config.nonprod.js, config.prod.js files which will replace with config.js during the pipeline.

### 2. Enable dynamic config loading at application runtime
👉 Add <script src="%PUBLIC_URL%/config.js"></script> to your ~/public/index.html file.

Explanation:


<!DOCTYPE html>
<html lang="en">
    <head>
        <link rel="shortcut icon" href="../favicon.ico" />
        <title>My React App</title>
    </head>
    <body>
        <noscript>You need to enable JavaScript to run this app.</noscript>
        <div id="root"></div>

        <script src="%PUBLIC_URL%/config.js"></script>
    </body>
</html>
In Line 11 <script src="%PUBLIC_URL%/config.js"></script>, we added config.js using a script tag enabling our React App to consume the configuration file.

The %PUBLIC_URL% will insert the URL of our app and the app will be able to get access to all the variables inside the config.js file.

### 3. How to access variables?
👉 These variables will go inside the window object. Hence, to access them, all we need to do is call window.VARIABLE_NAME and we are good to go.

Explanation:

e.g. In ~/public/config.js

```
# ~/public/config.js

var REACT_APP_API_ENDPOINT = 'https://apigw-nonprod.cevo.com.au/demo'
var REACT_APP_ROUTER_BASENAME = '/newServices';
```

Then you can access REACT_APP_ROUTER_BASENAME anywhere from your React App by using
```
# ~/src/app.tsx

type TAppProps = {};

const App: React.FC<TAppProps> = () => {
  return (
    <BrowserRouter basename={window.REACT_APP_ROUTER_BASENAME}>
      <Routes>
        <Route path="/" element={<Home />} />
      </Routes>
    </BrowserRouter>
  );
};
```
 
### 4. [CI] Build the code
👉 Let’s run npm run build / yarn build. It will not only compile and build our React App but also contains the config.js and moves env/ folder over to dist folder as well.
 
Explanation:

```
Before npm run build / yarn build:

my-app/
├─ public/
│  ├─ favicon.ico
│  ├─ index.html
|  ├─ config.js
│  ├─ env/
│  │  ├─ config.nonprod.js
│  │  ├─ config.prod.js
├─ src/
```

After npm run build / yarn build:

```
my-app/
├─ dist/
│  ├─ favicon.ico
│  ├─ index.html
|  ├─ config.js
│  ├─ env/
│  │  ├─ config.nonprod.js
│  │  ├─ config.prod.js
```

5. [CD] Replace the configuration file
👉 Finally we replace the ~/dist/config.js with ~/dist/envs/config.<environment>.js during the CD deployment process.

Explanation:

For example, during my CD pipeline as illustrated in the CD NonProd / Replace config.js from the pipeline design diagram above. You can have an automated step that runs a bash cp /dist/envs/config.nonprod.js ./dist/config.jswhich replaces the config file based on environments.

## Conclusion
Avoiding environment-specific builds is a common agreement in the DevSecOps world and today we achieved a simple solution for popular React Apps. I hope this has been a useful tutorial to the world of DevSecOps, modern CI/CD pipelines, and React best practices.

If you’d like help with setting up advanced CI/CD pipelines, having a version-controlled and organizational level pipelines' template, developing highly reusable pipelines, improving your teams' DevOps practice, or moving your workloads to the cloud, please contact us