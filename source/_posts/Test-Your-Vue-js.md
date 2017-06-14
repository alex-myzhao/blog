---
title: Test Your Vue.js Project
date: 2017-06-13 20:25:40
tags:
  - vue
photos: images/vuejs-title.png
---

覆盖率高的测试可以在很大程度上提高软件产品的可靠性，这学期学习了软件测试课程之后更是深有体会。本文介绍针对 Vue 框架进行单元测试和端到端测试的方法，以及通过 Travis CI 自动化这一过程从而进行持续集成。
<!--more-->
关于 [Vue.js](http://cn.vuejs.org/v2/guide/)，是一套构建用户界面的渐进式框架。类似 Angular.js，Vue.js 通过 MVVM 设计将界面、数据和操作逻辑解耦，易于上手并且方便使用，再加上其官方提供的多语言支持详尽文档和循序渐进的教程，在这一年的时间里非常热门。本文默认读者已经对这个框架有了相当程度对了解，如果正在寻找入门教程，强烈推荐按照顺序阅读[官方文档](http://cn.vuejs.org/v2/guide/)。
使用 vue-cli 生成带有单元测试模版的项目：

```bash
npm install -g vue-cli
vue init webpack <project-name>
```

分析生成的项目结构如下：

```
.
├── build/                      # webpack config files
│   └── ...
├── config/
│   ├── index.js                # main project config
│   └── ...
├── src/
│   ├── main.js                 # app entry file
│   ├── url-config.js           # server info
│   ├── App.vue                 # main app component
│   ├── lib/                    # useful js modules
│   ├── components/             # ui components
│   │   └── ...
│   ├── pages/                  # routered pages
│   │   └── ...
│   ├── store/                  # vuex support
│   │   ├── index.js
│   │   └── ...
│   └── assets/                 # module assets (processed by webpack)
│       └── ...
├── static/                     # pure static assets (directly copied)
├── test/
│   ├── e2e/                    # end-to-end test
│   └── unit/                   # unit test
├── .babelrc                    # babel config
├── .postcssrc.js               # postcss config
├── .eslintrc.js                # eslint config
├── .editorconfig               # editor config
├── .travis.yml                 # travis CI
├── index.html                  # index.html template
└── package.json                # build scripts and dependencies
```

## 单元测试 (Unit Test)

对于每一个vue组件，单元测试测试主要关注：
- 组件的数据模型
- 方法的输入输出
- 组件间的数据传递情况
- 异步操作

vue-cli 生成的项目通过 karma 来进行单元测试。在 `test/unit/spec/` 目录下的不同文件中，进行断言编写。比如可以为 `home.vue` 组件编写测试，判断项目名是否为 Go Movie：
```javascript
describe('home.vue', () => {
  it('should render correct contents', () => {
    const Constructor = Vue.extend(Home)
    const vm = new Constructor().$mount()
    expect(vm.$el.querySelector('.top-bar__app-name div').textContent)
      .to.equal('Go Movie')
  })
})
```
测试组件中的方法：先通过 `let vm = new Vue(app).$mount();` 获取到 vm，然后就可以通过 vm 直接调用组件的方法或者访问组件的属性了。
```javascript
it('test method', () => {
    let vm = new Vue(app).$mount();

    vm.setMessage('test');
    expect(vm.message).toEqual('test');
})
```
组件间的数据传递情况：使用`Vue.extend()`将组件挂载Vue构造器，将 props 传入然后测试子组件：
```javascript
function getRenderedVm (Component, propsData) {
    const vue = Vue.extend(Component)
    const vm = new vue({ propsData }).$mount()
    return vm
}
```
关于异步情况，可以使用`Vue.nextTick`查看异步数据更新后 dom 是否发生了变化
```javascript
Vue.nextTick(() => {
    let title = vm.$el.getElementsByTagName('h1')[0]
    expect(title.textContent).toEqual('APP')
    done();
})
```

编写好脚本后，通过 `npm run unit` 即可执行单元测试，可以看到测试通过的信息以及代码覆盖率（由于这里只是做了一个 demo 所以覆盖率很低），如下图所示：
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/unit-res.png)

## 端到端测试 (E2E Test)

端到端测试的方式通过测试模拟用户行为，打开浏览器自动化地做一些操作（如输入、点击等）看看程序是否能够达到预期等结果。
Vue.js 可以使用 nightwatch 工具进行端到端测试。Nightwatch 是基于 NodeJS 的 e2e 测试框架，通过发送 HTTP 请求到 Selenium WebDriver 来控制浏览器进行测试。如下图所示：
![](https://cdn.imyzf.com/img/blog/2016/vuejs-2-test-analysis/nightwatch.png)
Nightwatch 在 `nightwatch.conf.js` 文件中进行配置，默认占用端口4444。
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/vue-e2e.png)

然后在 `test/e2e/specs/` 目录下进行测试用例的编写。
```javascript
'default e2e tests': function (browser) {
  // automatically uses dev Server port from /config.index.js
  // default: http://localhost:8080
  // see nightwatch.conf.js
  const devServer = browser.globals.devServerURL

  browser
    .url(devServer)
    .waitForElementVisible('#app', 5000)
    .assert.elementPresent('.home')
    // .assert.containsText('h1', 'Hello Vue, this is home page')
    // .assert.elementCount('img', 1)
    .end()
}
```

通过执行 `npm run e2e` 进行端到端测试。测试结果如下：
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/e2e-res.png)

## Travis CI 集成

为了方便起见，我们可以通过一行脚本将两个测试串联到一起。`"test": "npm run unit && npm run e2e"`
然后通过 Travis CI 工具进行自动化构建以及测试。Travis CI 提供在线托管 CI 服务，对开源项目免费，并且 Travis CI 很容易部署：
1. 登录 [Travis CI 网站](https://travis-ci.org/) ![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/travis-1.png)
2. 激活需要 CI 的 GitHub 项目
3. 编写 `.travis.yml` 配置文件
4. 在 GitHub 上提交代码触发 （optional: 添加 build 状态图标）

实际操作的时候遇到了一些小问题，NodeJS 的项目需要指定 dist 为 trusty 并且在执行测试前完成一些指定才可以，否则端到端测试会因来不及启动而失败。
```yml
dist: trusty
addons:
  chrome: stable
language: node_js
node_js:
  - "7"
before_script:
  - "export DISPLAY=:99.0"
  - "sh -e /etc/init.d/xvfb start"
  - sleep 3 # give xvfb some time to start
branches:
  only:
  - master
```

Travis CI 自动化测试后状态如下：
![](http://oowu6eof3.bkt.clouddn.com/blog/docker-ci-cd/travis-res.png)

## Reference

- https://cn.vuejs.org/v2/guide/unit-testing.html
- http://www.jianshu.com/p/a515fbbdd1b2
