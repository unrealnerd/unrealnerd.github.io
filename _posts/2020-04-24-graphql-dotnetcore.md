---
layout: post
title: Graphql with dotnetcore
description: simple notes on graphql with dotnet core
author: bitsmonkey
tags: dotnetcore, graphql, api
---

What does GraphQL solve?

Overfetching, Underfetching, and reduced round trips. While using GraphQL we move the control of data over to API instead of the client-side. Also, GraphQL came to existence from the labs of Facebook while working on a mobile app.
These days servers have become so cheap it makes sense to move the control over to server instead of the client so the end-users consume the least amount of data also the processing power of his device.

Well, I must say this is kind of a genre which is created, there are scenarios where the other way is also true.

Writing a GraphQL with Dotnet core is as easy as this one.

![grpphql](/img/graphql-with-dotnetcore.jpg){:.coverimage}

Create a web API with default webapi template

`dotnet new webapi -o graphql-sample`

Add these two libraries, One is the GraphQL implementation and one more is to enable a cool library which makes testing and checking the API easier.

`dotnet add package GraphQL`

`dotnet add package graphiql`

Insert this line

`app.UseGraphiQl("/graphql");`

inside `Configure` method in `Startup.cs`. Which says expose the cool UI to test GraphQL at `/graphql` endpoint.

Add a new class `GraphQLQuery` which will be used to [model bind](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/model-binding?view=aspnetcore-3.1) our through our controllers. In our first sample, we will just use the Query property of this model to retrieve the query to run on the server.

```cs
    public class GraphQLQuery
    {
        public string OperationName { get; set; }
        public string NamedQuery { get; set; }
        public string Query { get; set; }
        public JObject Variables { get; set; }
    }
```

Now let us add our domain objects for which we will use the hardcoded list of movies from IMDB.

```cs
    public class Movie
    {
        public string Title { get; set; }
        public string Year { get; set; }
        public string Stars { get; set; }
        public float Rating { get; set; }
        public string Genre { get; set; }
    }
```

Like in C# we write models and methods in it to access the data, in GraphQL we use schemas to do the same.

Add this MovieSchema

```cs
  public class MovieSchema 
  {
    private ISchema Schema { get; set; }
    public ISchema GraphQLQuoteSchema 
    {  
      get 
      {
        return this.Schema;
      }
    }

    public MovieSchema() 
    {
      this.Schema = GraphQL.Types.Schema.For(@"
          type Movie {
            title: String,
            year: String,
            stars: String,
            rating: Float,
            genre: String,
          }

          type Query {
              movies: [Movie],
              moviesByGenre(genre: String): [Movie],
          }
      ", _ =>
      {
        _.Types.Include<Query>();
      });
    }

  }
```

If you look at the schema which is written string is the one which defines the GrahQL queries and the types used. We have defined 2 queries `movies` & `moviesByGenre(genre: String)`. The first query returns all the movie types and the latter get movies filtered with the genre passed.

Now to make our API understand these queries we create this below class `Query`

```cs
    [GraphQLMetadata("movies")]
    public IEnumerable<Movie> GetMovies()
    {
        return movies;
    }


    [GraphQLMetadata("moviesByGenre")]
    public IEnumerable<Movie> GetQuoteByRegion(string genre)
    {
        return movies.Where(q => q.Genre == genre);
    }
```

The attributes that you see on the top of the methods is what maps these methods to the queries defined in the schema. The `movies` is just a collection of data.

That is it!
When you run this project and browse the `/graphql` endpoint you would see the [GraphiQL](https://github.com/graphql/graphiql) UI browse the API.

In the editor now enter the below query one by and see it for your self.

Query1

```js
{
  moviesByGenre(genre:"Drama") {
    title,
    rating
  }
}
```

Query2

```js
{
  moviesByGenre(genre:"Drama") {
    title,
    rating
  }
}
```

For the full source code checkout this [GitHub Repo](https://github.com/unrealnerd/graphql-sample).


*Photo by Alina Grubnyak on Unsplash*
