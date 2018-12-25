## React首次渲染

>>> 前后端分离后，模版引擎从服务端过渡到了客户端，减轻了服务器的压力，
>>> 提高了页面速度，同时前端也作为一个技术岗位在研发中变的更加职业和重要。
>>> 前端可以通过工程化手段，解决横向和纵向的协作效率问题。

***言归正传，首次渲染到底什么鬼***
```
我们知道单页应用，可以理解就是无论你怎么和前端服务器交互，拿到的就是 一个HTML和一堆js文件。
在浏览器中键入地址拿到如上所说的html和js，单页应用框架提供一个入口render方法，根据状态渲
染视图，地址也是一种状态，所以不同路径，js会画出不同的视图页面，其实整体上说，单页页面路由
跳转，只是视图改变，永远就一个页面。
```

问题是首次渲染过程，需要加载html，js，执行js创建首个视图，在这个过程中，特别痛
苦的是需要用户等待，文件下载过程、js渲染整个骨架这两个过程很耗时。

解决方案如下？
+ CDN加速，静态资源存储到分布式节点上，解决了网络延迟问题。
+ 代码分割，按需加载。减小js文件的大小。
+ 文件压缩，开启gzip模式。

+ 增加骨架页，首次渲染不至于白屏。
+ 服务端渲染，虚拟DOM的理念，天生就可以在Nodejs平台上同构。
+ 预渲染，生成多页。

>>> **暂不讨论SEO需求场景！！！！**

如上诸多方案，前三条是解决网络传输耗时问题，后三条是解决首次渲染骨架耗时问题

增加骨架页很简单，大量公司都采用，但是只是优化体验，并没有实际解决耗时问题。
我们着重讨论

***服务端渲染*** 
***预渲染***

这两个方案

+ 服务端渲染 

   **服务端渲染，难免遇到一个问题就是高可用，如何实现高可用？建议提供固有的前端资源托管作为兜底方案**

   **服务端渲染，面临挑战是前端技术栈纵向更深如，前端基础建设完善、前端体系规范化的团队比较适用。**
   **而如果仅仅是几人小团队，首先javascript不是强类型，前端代码健壮性较差，服务不可用，不如不用**

   ##### express+react 
   [详情参见](https://github.com/xusai2014/FilmReview)
   此方案可以用来改造现有的前端框架解决骨架加载问题和SEO问题
   **该方案暂不支持代码分割的前端方案**
    
  
      import { renderToString } from 'react-dom/server';
      import { StaticRouter } from 'react-router-dom';
      import  rootReducer from '../client/reducers';
      import store from '../client/store';
      import RouterAll from '../client/routers';
      import { Provider } from "react-redux";
      
      const app = express();
      app.use('/pages/*',renderString);
      const storage = store()(rootReducer);
      function renderString(req, res) {
        // Render the component to a string
        const context = {}
        const html = renderToString(
          <Provider store={ storage } >
            <StaticRouter location={req.url} context={context}>
               <RouterAll />
            </StaticRouter>
          </Provider>
        );
        const preloadedState = storage.getState();
        // Send the rendered page back to the client
        res.send(renderFullPage(html, preloadedState))
      }

      function renderFullPage(html, preloadedState) {
        return `<html>
           <head>
               <script src="/node_modules/jquery/dist/jquery.js"></script>
               <script src="/node_modules/bootstrap/dist/js/bootstrap.min.js"></script>
           </head>
           <body>
           <div id="root">${html}</div>
           <script>
             // WARNING: See the following for security issues around embedding JSON in HTML:
             // http://redux.js.org/docs/recipes/ServerRendering.html#security-considerations
             window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState).replace(/</g, '\\u003c')}
           </script>
           
           
           <script src="https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js"></script>
           <script src="https://cdn.bootcss.com/jquery-cookie/1.4.1/jquery.cookie.js"></script>
           <script  src="/bin/client.bundle.js"></script>
           </body>
           </html>
        `
      }  

   ##### next.js 
   一个很流氓的大前端框架，它并不希望无缝的与现有前端框架结合，它就是一个前端框架，希望打造新的前端解决方案，next化。
   当然开发的技术理念没变，所以上手很简单。使用它也要适量考虑技术栈迁移成本。
   
   [详情参见](https://github.com/zeit/next.js)
   
    
      npm install --save next react react-dom
    
   很老的老套路，项目文件结构就是主要的API，也就是说基于你的文件结构，框架会自动绑定路由
    
    
    
      //After that, the file-system is the main API. Every .js file becomes a route that gets automatically processed and rendered.Populate ./pages/index.js inside your project:
      export default () => <div>Welcome to next.js!</div>
    
      npm run dev
    
   基于import声明，代码自动分割。实现按需加载。

+ 预渲染
    如果没有很深场景的SEO需求，预渲染对现有前后端分离改造很适合。
    
    基本原理是基于[无界面模式的浏览器node库](https://github.com/GoogleChrome/puppeteer)
    
    ```
        yarn add --dev react-snap
    ```
    package.json
    
    ```json
      "scripts": {
        "postbuild": "react-snap"
      },
      "reactSnap": {
          "source": "build",
          "port": "3300",
          "userAgent": "ReactSnap Android",
          "skipThirdPartyRequests": false,
          "asyncScriptTags": true,
          "cacheAjaxRequests": true,
          "minifyHtml": {
            "collapseWhitespace": false,
            "removeComments": false
          },
          "destination": "./build/dist", // 静态页面资源存储路径
          "include": [
            "/home/index",
          ]
        },
    ```
    
    ```javascript
      import { hydrate, render } from "react-dom";
      const rootElement = document.getElementById("root");
      if (rootElement.hasChildNodes()) {
        hydrate(<App />, rootElement);
      } else {
        render(<App />, rootElement);
      }
    ```
    部署方案, nginx配置
    
    [系统预装环境](https://github.com/GoogleChrome/puppeteer/blob/master/docs/troubleshooting.md)
    
    比如ubuntu
    ```
        gconf-service
        libasound2
        libatk1.0-0
        libatk-bridge2.0-0
        libc6
        libcairo2
        libcups2
        libdbus-1-3
        libexpat1
        libfontconfig1
        libgcc1
        libgconf-2-4
        libgdk-pixbuf2.0-0
        libglib2.0-0
        libgtk-3-0
        libnspr4
        libpango-1.0-0
        libpangocairo-1.0-0
        libstdc++6
        libx11-6
        libx11-xcb1
        libxcb1
        libxcomposite1
        libxcursor1
        libxdamage1
        libxext6
        libxfixes3
        libxi6
        libxrandr2
        libxrender1
        libxss1
        libxtst6
        ca-certificates
        fonts-liberation
        libappindicator1
        libnss3
        lsb-release
        xdg-utils
        wget

    ```
    
    ```
        location / {
            root /usr/local/nginx/build/dist; //根据具体服务器配置静态资源路径
            index index.html;
            try_files $uri $uri/ /200.html;
        }

    ```
    
    
    

