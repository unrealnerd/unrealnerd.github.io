---
layout: post
title: Dotnet core GraphQL with postgres 
description: Creating a graphql application with postgres as backend via efcore
author: bitsmonkey
tags: efcore, dotnet, postgres, graphql
---

![postgres-golang](/img/dotnetcore-postgres-graphql.jpg){:.coverimage}

Lets create a GraphQL application in `dotnet core 3.1` using [HotChoclate](https://github.com/ChilliCream/hotchocolate).

Creating a new web api project

`dotnet new web -n GraphApi`

This package mostly handles all the aspects of Graphql. Also important factor for choosing this library is it passes all the kitchen sink test, more details [here](https://chillicream.com/blog/2019/06/05/hot-chocolate-9.0.0)

`dotnet add package HotChocolate.AspNetCore --version 10.5.3`

Adding GraphQL [playground IDE](https://github.com/graphql/graphql-playground) for easing our development.

`dotnet add package HotChocolate.AspNetCore.Playground --version 10.5.3`

One last dependency is the the postgres efcore

`dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 3.1.4`

The Pi version 😄.

Lets set thing in `Startup.cs` now where all the middleware is registered.

```cs
services.AddScoped<DummyService>();

services.AddDbContext<ApplicationDbContext>(options =>
	options.UseNpgsql("User Id=username; Password=password; Server=dbname.host.com; Port=7530 ; Database=dbname; Ssl Mode=Require; Server Compatibility Mode=Redshift",
	builder => builder.ProvideClientCertificatesCallback(cert =>
	{
		cert.Add(
			new System.Security.Cryptography.X509Certificates.X509Certificate("./Cert/cert.pem")
			);
	}
)));
```
The first class registered is something that we will create later.

Add the above in `ConfigureServices` method. Ignore the `Ssl Mode=Require; Server Compatibility Mode=Redshift` part in the connection string and the builder action, if you are not using SSL connection.

If using SSL connection note the `Server Compatibility Mode=Redshift` this is something you might get stuck with no much details on the error message. You will some error message like this one `Unsupported startup parameter: extra_float_digits`. More details on this [official documentation](http://www.npgsql.org/doc/compatibility.html#amazon-redshift).


```cs
services.AddGraphQL(s => SchemaBuilder.New()
                                    .AddServices(s)
                                    .AddType<DummyType>()
                                    .AddQueryType<Query>()
                                    .Create());
```

This is mostly all the HotChocolate registration where you are giving HotChocolate the details required to create schema, types and queries.

Now inside `Configure` method we will use the registered,

```cs
if (env.IsDevelopment())
{
	app.UseDeveloperExceptionPage();
	app.UsePlayground();
}

app.UseGraphQL("");
```
You business model will have a partnering model which is used to expose only the fields you want to the graphql queries as follows,

```cs
public class Dummy
{
	public long Id { get; set; }
	public string DummyProperty { get; set; }
}

public class DummyType : ObjectType<Quote>
{
	protected override void Configure(IObjectTypeDescriptor<Quote> descriptor)
	{
		descriptor.Field(x => x.DummyProperty).Type<StringType>();
	}

}
```

We have exposed only one property for pur business model `Dummy`. This represets the Type part of the Graphql. For the query node we will create a new class `Query`

```cs
public class Query
{
	private readonly DummyService _dummyService;

	public Query(DummyService dummyService)
	{
		_dummyService = dummyService;
	}

	public IQueryable<Dummy> DummyList => _dummyService.GetSomeData();

}
```

`DummyService` is just an abstraction for the efcore repository layer you can also inject `DbContext` directly if required.

The Efcore is something which is explained in here before so just paste code and skip to running the app part.

```cs

public class RepositoryContext : DbContext
{
	public DbSet<Dummy> DummyDbSet { get; set; }

	public ApplicationDbContext(DbContextOptions<RepositoryContext> options)
			: base(options)
	{
	}

	protected override void OnModelCreating(ModelBuilder modelBuilder)
	{
		modelBuilder.Entity<Dummy>(entity =>
		{
			entity.ToTable("tablename");

			entity.Property(x => x.Id).HasColumnName("Id");
			entity.Property(x => x.QuoteNumber).HasColumnName("Column1");

			entity.HasKey(x=>x.Id);
		});
	}
}

public class DummyService
{
	private readonly RepositoryContext _dbContext;

	public IQueryable<Dummy> GetSomeData() =>
			_dbContext.DummyDbSet;

	public DummyService(RepositoryContext dbContext)
	{
		_dbContext = dbContext;
	}
}
```

We are all set once we run this application we go to `http:\\localhost:5000\playground`, which opens up the GraphQL IDE.

run this query and it will list all the data.

```gql
dummyList{
	id
}
```

Enabling paging is as easy as adding this attribute in `Query`.

```cs
[UsePaging(SchemaType = typeof(QuoteType))]
public IQueryable<Dummy> DummyList => _dummyService.GetSomeData();
```

Explore away!

Photo by [Praewthida K](https://unsplash.com/s/photos/pattern?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)