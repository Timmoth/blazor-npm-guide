Using NPM packages in a Blazor Wasm app
=======================================

Find the demo project on [Github](https://github.com/Timmoth/blazor-npm-guide)  
  
Prerequisites:
--------------------------------------------------------------------------------------------------

*   [Nodejs](https://nodejs.org/en/download/)
*   [Visual Studio 2022](https://visualstudio.microsoft.com/)
*   ASP.NET and web development workload

Setup your solution and blazor wasm project

    dotnet new blazorwasm -n blazor-npm-guide
    dotnet new sln
    dotnet sln add ./blazor-npm-guide
    

Create a new directory for our npm packages and the js source that makes use of them

    mkdir jslib
    cd jslib
    mkdir src
    

Initialize a new npm project (you can just skip past all the questions)

    npm init
    

Install webpack to bundle our library and put the output inside wwwroot/js

    npm install webpack webpack-cli
    

Create the webpack.config.js

    const path = require("path");
    
    module.exports = {
        module: {
            rules: [
                {
                    test: /\.(js|jsx)$/,
                    exclude: /node_modules/,
                    use: {
                        loader: "babel-loader"
                    }
                }
            ]
        },
        output: {
            path: path.resolve(__dirname, '../wwwroot/js'),
            filename: "jslib.js",
            library: "jslib"
        }
    };

Add a script to build webpack in package.json

    {
      "name": "jslib",
      "version": "1.0.0",
      "description": "",
      "main": "webpack.config.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "build": "webpack --mode production"
      },
      "author": "",
      "license": "ISC",
      "dependencies": {
        "highlight.js": "^11.4.0",
        "webpack": "^5.68.0",
        "webpack-cli": "^4.9.2"
      },
      "devDependencies": {
        "@babel/core": "^7.17.0",
        "babel-loader": "^8.2.3"
      }
    }
    

Install babel

    npm install babel-loader @babel/core --save-dev
    

Install your npm package, in this instance we're going to install highlight.js

    npm install highlight.js
    

Create src/highlight\_lib.js

    import hljs from 'highlight.js';
    
    export function highlight() {
        hljs.highlightAll();
    }

Create src/index.js to act as the root that exposes the functionality of our npm packages

    import { highlight } from './highlight_lib';
    
    export function Highlight() {
        return highlight();
    }

Build our library

    npm run build
    

Npm will have built our library and placed it in wwwroot/js, make sure to include the script in the body of the index.html

    
    
        <!DOCTYPE html>
    <html lang="en">
    
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
        <title>Timmoth</title>
        <base href="/" />
        <link href="css/bootstrap/bootstrap.min.css" rel="stylesheet" />
        <link href="css/app.css" rel="stylesheet" />
        <link href="webclient.styles.css" rel="stylesheet" />
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.4.0/styles/default.min.css">
    </head>
    
    <body>
        <div id="app">Loading...</div>
    
        <div id="blazor-error-ui">
            An unhandled error has occurred.
            <a href="" class="reload">Reload</a>
            <a class="dismiss">🗙</a>
        </div>
        <script src="_framework/blazor.webassembly.js"></script>
        <script src="js/js-lib.js"></script>
    
    </body>
    
    </html>
    

Inject IJSRuntime & make jsinterop call to jslib.Highlight

    
    @page "/"
    @inject IJSRuntime JsRunTime
    
    <pre>
    <code>import { highlight } from './highlight_lib';
    
    export function Highlight() {
        return highlight();
    }</code></pre>
    
    @code
    {
        protected override async Task OnAfterRenderAsync(bool firstRender)
        {
            if (firstRender)
            {
                await JsRunTime.InvokeAsync<string>("jslib.Highlight");
            }
            await base.OnAfterRenderAsync(firstRender);
        }
    }
