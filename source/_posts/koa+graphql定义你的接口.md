---
title: koa + graphql的使用
date: 2020-08-16 13:13:14
---

#### 目标
> 完成http请求到graphql的请求改造

#### 实现
> 内容大部分来自己graphql与apollo-server官网, 内部加上自己的一些理解,至于改造嘛,就自己动手咯
> 

```javascript
const Koa = require("koa");
const { ApolloServer, gql } = require("apollo-server-koa");
const GraphQLJSON = require("graphql-type-json");
const { GraphQLScalarType } = require("graphql");
const { Kind } = require("graphql/language");
const books = [
  {
    title: "Harry Potter and the Chamber of Secrets",
    author: "J.K. Rowling",
  },
  {
    title: "Jurassic Park",
    author: "Michael Crichton",
  },
];

const users = [
  {
    id: "1",
    name: "Elizabeth Bennet",
  },
  {
    id: "2",
    name: "Fitzwilliam Darcy",
  },
];

const typeDefs = gql`
  type Query {
    #   返回字符串
    hello: String
    # 返回数组
    books: [Book]
    # 通过参数查询
    user(id: ID): User
    info(date: Date): String
    # 读取数据
    foo: Foo
  }
  #使用第三方定义的标量 json类型
  scalar JSON
    # 使用自定义的date类型
  scalar Date
  type Foo {
    user: JSON
    created: Date
  }

  type Book {
    title: String
    author: String
  }

  type User {
    id: ID
    name: String
  }
`;

const resolvers = {
  JSON: GraphQLJSON, // 使用第三方的json标量
  Date: new GraphQLScalarType({
    // 自定义标量
    name: "Date",
    description: "Date custom scalar type",
    parseValue(value) {
      // variables 参数解析路径
      return new Date(value);
    },
    serialize(value) {
      return value.getTime();
    },
    parseLiteral(ast) {
      // 字面量参数路径
      if (ast.kind === Kind.INT) {
        // ast.value 永远是 字符串类型 所以需要转换
        return new Date(+ast.value);
      }
      return null;
    },
  }),
  Query: {
    hello: () => {
      return "Hello shaokun!";
    },
    books: () => books, // 直接返回数据库中的books
    user(parent, args, context) {
      // parent 连续查询 中 上一次查询的结果
      // args 查询参数 如上面的id
      // context 全局数据的拿取
      console.log(parent, args, context.name, context.age);
      return users.find((user) => user.id === args.id);
    },
    foo: () => {
      return {
        user: { name: "shaokum" },   // 如果返回的不是json类型 则为null
        created: new Date(),        // Date类型 实际返回给前端的是 定义的scalar中的serialize方法
      };
    },
    info: (_, args) => {
      // 查询方式 字面常量方式查询
      // {
      //     info(data:1597552212000)
      // }
      console.log("---info-", args); // 通过Date.parseLiteral 方法拿到解析的数据
      return "this date is " + args.date.toString();
    },
  },
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: async (request) => {
      // 可获取到rquest请求
    // 可以全局挂载,如授权token db实例
    return {
      name: "shaokun",
      age: 18,
    };
  },
});

const app = new Koa();
server.applyMiddleware({ app });

app.listen({ port: 4000 }, () =>
  console.log(`🚀 Server ready at http://localhost:4000${server.graphqlPath}`)
);

```

##### 结果演示

![graphql query demo](/react/graphql.gif) 

##### 关于我
区块链技术痴迷的程序猿一枚，如果你喜欢我的文章，可以加上微信共同学习，共同进步。  



