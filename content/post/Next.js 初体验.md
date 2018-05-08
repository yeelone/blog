\---

title: "Next.js 初体验"

date: 2018-05-08T15:44:29+08:00

tags : ["Next.js", "typescript"]

categories : ["Next.js"]

\---



#Next.js 初体验

## 安装 Next.js

```shell
$ yarn add next react react-dom

$ yarn run dev //运行查看是否成功 
```

##配置 typescript && less 

```
$ yarn add @zeit/next-typescript @zeit/next-less typescript less
```

###创建 next.config.js

```javascript
// next.config.js
const withTypescript = require('@zeit/next-typescript')
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin')

module.exports = withTypescript({
  webpack(config, options) {
      // Do not run type checking twice:
    if (options.isServer) config.plugins.push(new ForkTsCheckerWebpackPlugin())
    return config
  },
  typescriptLoaderOptions: {
    transpileOnly: false
  }
})
```

###创建 tsconfig.json

```
{
  "compileOnSave": false,
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "jsx": "preserve",
    "allowJs": true,
    "moduleResolution": "node",
    "allowSyntheticDefaultImports": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "removeComments": false,
    "preserveConstEnums": true,
    "sourceMap": true,
    "skipLibCheck": true,
    "baseUrl": ".",
    "lib": [
      "dom",
      "es2016"
    ]
  }
}
```

### 修改pages/index.tsx 

```
export default () => <div>{getName("elone")}</div>

function getName(person: string) {
    return "Hello, " + person;
}

```

