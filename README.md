# node_graphql_example

# Refererence

[https://www.youtube.com/watch?v=ZQL7tL2S0oQ](https://www.youtube.com/watch?v=ZQL7tL2S0oQ)

# GraphQL

- You compose the query exactly the way you want.
- You create a query which contains information for the data you want.
- You don't need to call API multiple times.
- All we have to do is to define the object for getting data from server.
- Typing out Objects is really what GraphQL is all about.

### Basic Implementation Requirements

1. Import Libraries 
2. Define Type
3. Define Root Query
4. Define Schema (with Root Query)

### GraphQL example

```json
query {
    authors {
        name 
        books {
            name     
        }
    }
}
```

### GraphQL with argument

```json
query {
  author(id: 1) {
        name
  }
}
```

### GraphQL mutation

```json
mutation {
  addBook(name: "New Name", authorId: 4) {
    name
  }
}
```

### npm

```
npm i express express-graphql graphql --save
```

### Basic Server

```jsx
const express = require('express');

/* import libraries */
const expressGraphQL = require('express-graphql');
const {
    GraphQLSchema,
    GraphQLObjectType,
    GraphQLString
} = require('graphql');

const app = express();

/* Define Schema */
const schema = new GraphQLSchema({
   query: new GraphQLObjectType({
       name: 'HelloWorld',
       fields: () => ({
           message: {
               type: GraphQLString,
               resolve: () => 'Hello World'
           }
       })
   })
});

/* router with `graphiql` */
app.use('/graphql', expressGraphQL({
    schema: schema,
    graphiql: true  //Webbrowser에 UIInterface 제공
}));

app.listen(5000, () => {
    console.log('Server Running');
});
```

### Entire Code

```java
const express = require('express');
const app = express();

/* import libraries */
const expressGraphQL = require('express-graphql');
const {
    GraphQLSchema,
    GraphQLObjectType,
    GraphQLString,
    GraphQLList,
    GraphQLInt,
    GraphQLNonNull
} = require('graphql');

//Demo Data
const authors = [
    {id: 1, name: 'J. K. Rowling'},
    {id: 3, name: 'J. R. R. Tolien'},
    {id: 4, name: 'J. R. R. Tolien'}
];

//Demo Data
const books = [
    {id: 1, name: 'Harry Potter and the Cahmber Of Secrets', authorId: 1},
    {id: 2, name: 'Harry Potter and the Cahmber Of Secrets', authorId: 1},
    {id: 3, name: 'Harry Potter and the Cahmber Of Secrets', authorId: 3},
    {id: 4, name: 'Harry Potter and the Cahmber Of Secrets', authorId: 1},
    {id: 5, name: 'Harry Potter and the Cahmber Of Secrets', authorId: 4},
    {id: 6, name: 'Harry Potter and the Cahmber Of Secrets', authorId: 4},
    {id: 7, name: 'Harry Potter and the Cahmber Of Secrets', authorId: 1}
];

/* Define Type */
const BookType = new GraphQLObjectType({
    name: 'Book',
    description: 'This represents a book written by an author',
    fields: () => ({
        id: {type: GraphQLNonNull(GraphQLInt)},
        name: {type: GraphQLNonNull(GraphQLString)},
        authorId: {type: GraphQLNonNull(GraphQLInt)},
        author: { //Nested Object
            type: AuthorType,
            resolve: (book) => { //here argument returns `parent object` which is `book` here.
                //Let's assume that authors are data from Server
                return authors.find(author => {
                    return author.id === book.authorId
                })
            }
        }
    })
});

/* Define Type */
const AuthorType = new GraphQLObjectType({
    name: 'Author',
    description: 'This represents an author of a book',
    fields: () => ({
        id: {type: GraphQLNonNull(GraphQLInt)},
        name: {type: GraphQLNonNull(GraphQLString)},
        books: {
            type: GraphQLList(BookType),
            resolve: (author) => {
                //Let's assume that books are data from Server
                return books.filter(book => book.author.id === author.id);
            }
        }
    })
});

/* Define Root Query */
const RootQueryType = new GraphQLObjectType({
    name: 'Query',
    description: 'Root Query',
    fields: () => ({
        book: {
            type: BookType,
            description: 'A single Book',
            args: { //resolve 함수의 두 번째 argument의 타입을 정의할 수 있다.
                id: {type: GraphQLInt}
            },
            /*
                query {
                  book(id: 1) {
                    name
                  }
                }
            */
            //args에 정의한 내용을 바탕으로 data를 가져온다
            //args같은 경우는, graphQL에서 query를 던질 때 추가되는 부분임.
            resolve: (parent, args) => books.find(book => book.id === args.id)
        },
        books: {
            type: GraphQLList(BookType),
            description: 'List of All Books',
            resolve: () => books
        },
        author: {
            type: AuthorType,
            description: 'A Single Author',
            args: {
                id: {type: GraphQLInt}
            },
            resolve: (parent, args) => authors.find(author => author.id === args.id)
        },
        authors: {
            type: GraphQLList(AuthorType),
            description: 'List of All Authors',
            resolve: () => authors
        }
    })
});

/* Define Mutation Type for more functionality */
const RootMutationType = new GraphQLObjectType({
    name: 'Mutation',
    description: 'Root Mutation',
    fields: () => ({
        addBook: {
            type: BookType,
            description: 'Add a book',
            /* Define Args */
            args: {
                name: {type: GraphQLNonNull(GraphQLString)},
                authorId: {type: GraphQLNonNull(GraphQLInt)}
            },
            resolve: (parent, args) => {
                const book = { id: books.length + 1, name: args.name, authorId: args.authorId };
                books.push(book); //Let's assume that we are saving data into Database
                return book;
            }
        },
        addAuthor: {
            type: AuthorType,
            description: 'Add an Author',
            /* Define Args */
            args: {
                name: {type: GraphQLNonNull(GraphQLString)}
            },
            resolve: (parent, args) => {
                const author = { id: authors.length + 1, name: args.name };
                authors.push(author); //Let's assume that we are saving data into Database
                return author;
            }
        }
    })
});

/* Define GraphQL Schema with Root Query */
const schema = new GraphQLSchema({
    query: RootQueryType,
    /* Add Mutation Type */
    mutation: RootMutationType
});

/* router with `graphiql` */
app.use('/graphql', expressGraphQL({
    schema: schema,
    graphiql: true  //Webbrowser에 UIInterface 제공
}));

app.listen(5000, () => {
    console.log('Server Running');
});
```
