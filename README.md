## scaffold-react-ssr

react server side render 脚手架，抽象团队项目中的公共部件，沉淀技术方案实践，与技术规范。

### 组件书写规范

组件有两种写法，一种是函数(function)，另一种是类(class)。先看下函数的写法规范

```js
// eslint-disable-next-line no-use-before-define
Square.propTypes = {
  onClick: func.isRequired,
  value: string
};

// eslint-disable-next-line no-use-before-define
Square.defaultProps = {
  value: ""
};

export default function Square({ onClick, value = "" }) {
  return (
    <React.Fragment>
      <button className="square" onClick={onClick}>
        {value}
      </button>
      <style jsx>
        {`
          .square {
            background: #fff;
            border: 1px solid #999;
            float: left;
            font-size: 24px;
            font-weight: bold;
            line-height: 34px;
            height: 34px;
            margin-right: -1px;
            margin-top: -2px;
            padding: 0;
            text-align: center;
            width: 34px;
          }

          .square:focus {
            outline: none;
          }
        `}
      </style>
    </React.Fragment>
  );
}
```

上述函数写法，首先把 props 类型说明(propTypes)和 props 默认值(defaultProps)放在开头，让使用者直接知道能给组件传递什么 props;

### model 的使用

在参考 dva model 的基础上实现，简化了之前项目中 reducers 的编写。我们以一个具体的 model 实现来讲解：

```js
import Model from "../Model";

export default Model.getInstance(
  class extends Model {
    namespace = "TicTacToe";

    state = {
      history: [
        {
          squares: Array(9).fill(null)
        }
      ],
      stepNumber: 0,
      xIsNext: true,
      historySelectText: ""
    };

    actions = {
      async handleClickWithout(index) {
        return await new Promise((resolve, reject) => {
          setTimeout(() => {
            this.dispatch({
              type: "TicTacToe/handleClick",
              payload: index
            });
            resolve("complete");
          }, 1000);
        });
      }
    };

    reducers = {
      handleClick(state, { payload: index }) {
        const history = state.history.slice(0, state.stepNumber + 1);
        const current = history[history.length - 1];
        const squares = current.squares.slice();
        if (calculateWinner(squares) || squares[index]) {
          return state;
        }
        squares[index] = state.xIsNext ? "X" : "O";
        return {
          ...state,
          history: history.concat([
            {
              squares
            }
          ]),
          stepNumber: history.length,
          xIsNext: !state.xIsNext
        };
      }
    };
  }
);
```

代码第一行 import 的 Model 类是我们的基础类，所有 model 的实现都要继承自它，就像第三行代码的`class extends Model`。`Model.getInstance`是 Model 的一个静态方法，用来实例化你的 model 类。

继续看下类中的定义，namespace 定义命名空间，用以区分其他的 model。state 是 model 状态的定义。异步 action function 需要在 actions 中定义。reducers，顾名思义，是用来定义 redcuer 的，另外同步的 action function 会自动根据 reducer 生成。比如，handleClick 这个 recuder,相应会有一个 handleClick 名称的 action，该 action 在`Model.getInstance`实例化的对象中。

在页面组件中，使用 redux connect 函数连接 model 实例的 actions，就能实现在 view 中发送 action 到 store; 另外与页面相关 store 状态的注入，使用`namespace`定义的名称来获取,因为与页面相关状态数据被存储在 store state 对象的`namespace`空间上，例如：

```js
@connect(
  ({ TicTacToe }) => {
    return { ...TicTacToe };
  },
  {
    ...ticTacToeModel.actions
  }
)
```

### styled-jsx 样式书写

这次采用的是 nextjs 推荐的[styled-jsx](https://github.com/zeit/styled-jsx)样式方案，它属于 css in js 的一种实现，另外我们使 styled-jsx 集成了 sass。

现在来一块完成下样式的编写,假设我们需要写一个首页，先写首页组件：

```js
import React, { Component } from "react";
import styles from "./Index.scss";

class Index extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Welcome to React</h1>
        </header>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
        <style jsx>{styles}</style>
      </div>
    );
  }
}

export default Index;
```

`Index.scss`文件内容如下

```scss
/* @styled-jsx=scoped */

.App {
  text-align: center;
}

.App-logo {
  height: 80px;
}

.App-header {
  background-color: #222;
  height: 150px;
  padding: 20px;
  color: white;
}

.App-title {
  font-size: 1.5em;
  color: #fff;
}

.App-intro {
  font-size: large;
}
```

组件的样式你可以`import`他们来自 scss 文件，并使用一个`<style jsx>`标签渲染他们，就像以上代码中的`<style jsx>{styles}</style>`。如果你想要你的样式成为全局样式，记得添加 global 属性`<style jsx global>`。当然你也可以使用一次性全局选择器`:global()`,在样式文件中局部定义全局样式。详见:https://github.com/zeit/styled-jsx

### 接口请求 fetch 的封装

### 与 gee-ui 组件库集成开发

[gee-ui](https://github.com/geetemp/gee-ui)组件库是我们 geetemp 前端团队内部的组件库实现，目前建立在对 antd 的继承实现上。目标是
从各个项目中抽取出公共组件的实现，与设计人员合作，形成一套公共的组件规范，以期服务于各类 pc 端项目产品。

借助`yarn link`或`npm link`命令，脚手架与 gee-ui 可以轻松集成，同时开发。

##### 安装 gee-ui

从 github 拉取 gee-ui 并安装

```
git clone git@github.com:geetemp/gee-ui.git
yarn install
yarn dev
```

##### 集成 gee-ui

在 gee-ui 项目根目录下 link

```
cd gee-ui
yarn link
```

使用该脚手架的项目根目录下 link gee-ui,并启动项目进行开发

```
cd {项目根目录}
yarn link gee-ui
yarn start
```

### 技术栈

- react
- react-router
- redux
- styled-jsx
