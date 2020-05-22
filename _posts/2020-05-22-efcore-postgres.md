---
layout: post
title: Quick notes on efcore for postgres
description: quick notes on efcore for postgres
author: bitsmonkey
tags: efcore, postgres, dotnetcore 
---
![experiment](/img/efcore-postgres.jpg){:.coverimage}

Add the postgres ef library to your project from [Nuget.org](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL/)

    `dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL --version 3.1.3`

Setting the efocre middleware on `Startup.cs`, and picking the connection string from app settings

    ```csharp
    services.AddDbContext<DdlContext>(options =>
                options.UseNpgsql(Configuration.GetConnectionString("MoviesConnection"))
            );
    ```

Now Add Your ContextClass `MoivesContext`

    ```csharp
    public class DdlContext : DbContext
    {
        public DbSet<Movie> Movies { get; set; }

        public DdlContext(DbContextOptions<DdlContext> options) : base(options)
        {

        }       
    }
    ```

In case you have an existingDB which has table and column names different form your Models you can override modelling

    ```csharp
    protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Movie>(x =>
                {
                    x.ToTable("movies_new");


                    x.Property(y => y.Title).HasColumnName("movie_name");
                    x.Property(y => y.Year).HasColumnName("movie_year");
                    x.Property(y => y.Rating).HasColumnName("movie_rating");
                }
            );

        }
    ```



*Photo by Nam Anh on Unsplash*
