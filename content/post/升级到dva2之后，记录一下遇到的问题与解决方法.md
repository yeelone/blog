---
title: "升级到dva2之后，记录一下遇到的问题与解决方法"
date: 2018-03-21T22:44:29+08:00
tags : ["dva2", "react","redux"]
categories : ["react"]
---

### 升级到dva2之后，记录一下遇到的问题与解决方法 

##### 1、路由改动

```javascript
import React from 'react'
import PropTypes from 'prop-types'
import { Router, Route, IndexRoute } from 'dva/router'
import AdminApp from './routes/app'

 const cached = {}
 const registerModel = (app, model) => {
   if (!cached[model.namespace]) {
     app.model(model)
     cached[model.namespace] = 1
   }
}
 
 const Routers = function ({ history, app }) {
   const routes = [
     {
       path: '/admin',
       component: AdminApp,
       getIndexRoute (nextState, cb) {
         require.ensure([], require => {
           cb(null, { component: require('./routes/dashboard/') })
         }, 'dashboard')
       },
   ]

   return <Router history={history} routes={routes} />
 }
     
```

改成：

```javascript
import React from 'react'
import PropTypes from 'prop-types'
import { Router, Switch, Route } from 'dva/router' 
import dynamic from 'dva/dynamic' 

function RouterConfig({ history, app }) {
  const IndexPage = dynamic({
         app,
         component: () => require('./routes/app'),
       }) 

  const Dashboard = dynamic({
    app,
    component: () => require('./routes/dashboard/'),
  }) 
 })
 
 return (
        <Router history={history}>
          <Switch>
            <Route exact path="/" component={IndexPage} />
            <Route exact path="/admin/dashboard" component={Dashboard} />
          </Switch>
        </Router>
      ) 
} 
```

##### 2、 model 改动

由：

```javascript
subscriptions: {
    setup ({ dispatch, history }) {
      history.listen(location => {
        if (location.pathname === '/admin/questions'  ) {
          dispatch({
            type: 'query',
            payload: location.query,
          })
        }
      })
    },
  },
```



```javascript
 subscriptions: {
    setup ({ dispatch, history }) {
      history.listen( ({pathname,search}) => {
        var query = queryString.parse(search)

        if (pathname === '/admin/questions'  ) {
          dispatch({
            type: 'query',
            payload: query,
          })
        }
      })
    },
  },
  
```



3、 routerRedux.push进行跳转无法传参

```javascript
onPageChange (page) {
      dispatch(routerRedux.push({
        pathname,
        query: {
          ...query,
          page: page.current,
          pageSize: page.pageSize
        },
      }))
    },
    
```

具体改动内容请参考：https://github.com/ReactTraining/history/blob/v3/docs/Location.md 



```javascript
 onPageChange (page) {
       var query = {
            ...queryString.parse(location.search),
            page: page.current,
            pageSize: page.pageSize
        }
        dispatch(routerRedux.push({
            ...location,
            search:'?'+queryString.stringify(query),
        }))
    },
```

