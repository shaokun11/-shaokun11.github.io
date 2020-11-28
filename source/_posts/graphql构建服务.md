---
title: koa + graphql的使用(2)
date: 2020-11-15 22:35:14
---

#### 目标
> [koa + graphql的使用](https://shaokun11.github.io/2020/08/16/koa+graphql%E5%AE%9A%E4%B9%89%E4%BD%A0%E7%9A%84%E6%8E%A5%E5%8F%A3/)在这篇文章中,我们只实现了基本上的如何使用graphql,那么我们来实现业务上的graphql

#### 实现
- 目录结构,大体上分为三个目录结构,schema用于定于所有的类型,resolver定义相应类型的实现,models定义一些service的具体实现,如查询数据库,app.ts为项目入口

```bash
	./src:
	app.ts		models		resolvers	schema
	
	./src/models:
	index.ts
	
	./src/resolvers:
	index.ts	message.ts	user.ts
	
	./src/schema:
	index.ts	message.ts	user.ts
```

- 项目入口app.ts,可以看到和上诉定义的目录结构一致,引入根数据,注意其中的context,这里用于挂在挂载全局变量,实现一些中间件的挂载,如auth

```javascript
import Koa from "koa";
import { ApolloServer } from "apollo-server-koa";
import resolvers from "./resolvers";
import typeDefs from "./schema";
import models from "./models";
const app = new Koa();
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: {
    models,
    me: models.users[1],
  },
});
server.applyMiddleware({ app });

app.listen({ port: 5000 }, () =>
  console.log(`🚀 Server ready at http://localhost:4000${server.graphqlPath}`)
);

```

- 定义user的schema,注意其中的query加上了extend字段(message.ts内容一致)

```javascript

import {gql} from "apollo-server-koa";

const User = gql`
    extend type Query {
        users: [User!]
        user(id: ID!): User
        me: String!
    }

    type User {
        id: ID!
        username: String!
        messages: [Message!]
    }
`;
export default User;
```   

- 定义schema的导出文件,定义空的schema,以便具体的实体可以继承,固定写法

```javascritp
	import { gql } from "apollo-server-koa";
	import message from "./message";
	import user from "./user";
	const linkSchema = gql`
	  type Query {
	    _: Boolean
	  }
	
	  type Mutation {
	    _: Boolean
	  }
	
	  type Subscription {
	    _: Boolean
	  }
	`;
	
	export default [linkSchema, user, message];

```

- user的resolver具体实现,根据user的schema实现,这里注意resolver接收四个参数,根据官网的说法,我们一般用上前三个就够了,
* 分别是parent,即上一个查询的的结果
* args,本次执行的前端页面传过来的参数
* context,即app中的context,即返回的那个object

```javascript
import {IResolvers} from "apollo-server-koa";

const user: IResolvers = {
	Query: {
		users: (parent, args, {models}) => {
			return Object.values(models.users);
		},
		user: (parent, {id}, {models}) => {
			return models.users[id];
		},
	},
	User: {
		messages: (user, args, {models}) => {
			return Object.values(models.messages).filter(
				//@ts-ignore
				(message) => message.userId === user.id
			);
		},
	},
};

export default user;

```
- resolver 导出文件,直接导出数组即可  

```javascript
	import message from "./message";
	import user from "./user";
	
	export default [message, user];
```

- models 模拟本次程序的数据,现实中用数据库代替即可

```javascript
let users = {
  1: {
    id: "1",
    username: "hello shoakun 1",
    messageIds: [1],
  },
  2: {
    id: "2",
    username: "hello shaokun2",
    messageIds: [2],
  },
};

let messages = {
  1: {
    id: "1",
    text: "Hello World",
    userId: "1",
  },
  2: {
    id: "2",
    text: "By World",
    userId: "2",
  },
};

export default {
  users,
  messages,
};

```

##### 结果演示
[源码](https://github.com/shaokun11/gql-server)
![graphql query demo](/react/graphql2.gif) 

##### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  



