---
layout: post
title: Yet another cli with golang using cobra 
description: Building a cli tool using cobra framework in goland 
author: bitsmonkey
tags: golang, cobra, cli, go 
---

![cobra-golang-cli](/img/cobra-golang-cli.jpg){:.coverimage}

Getting work done the plain vanilla way is doing it in cli so I ended up here. Dont you love docker cli, github cli. who would'nt? most of the new cli's are built using golang [cobra framework](https://github.com/spf13/cobra).

Let's try this out. In this example I will try to build a cli which will talk to DB like getting schema information from a postgres DB.
For starters I recommend using [Cobra Generators](https://github.com/spf13/cobra#using-the-cobra-generator). 
Make sure if you are designing something production grade follow this simple quote from the cobra readme

> Commands represent actions, Args are things and Flags are modifiers for those actions.

This sample is just to touch up upon basics of what is required to start with Cobra cli. Here is the final project structure for future references

```sh
cmd/      # all the command implementions are here     
config/   # configuration related to this app like connection string etc,.
go.mod    # go module
go.sum    # go dependencies haskeys
handlers/ # command from the cmd talk to handlers which does the actual call the services
main.go   # applications starting point
repo/     # all the services implementation in our case its a service talking to the DB so name repo
```

All the command start from the `rootCmd` root command. and it can have any number of child or grand child. Here we will create root-->list-->table and root-->list-->schema, so list has 2 children and theier grand father is root.

Lets use [Cobra code generator](https://github.com/spf13/cobra/blob/master/cobra/README.md) to create commands its mostly auto fills a lot of things for us here is how it looks.

`go get github.com/spf13/cobra/cobra`

`cobra init [app]`

`cobra add list`

`cobra add list -p 'tableCmd'`

```go
var tableCmd = &cobra.Command{
	Use:   "table",
	Short: "list tables in schema",
	Long: `	list tables <schema> - will list the tables in the specified schema
			--limit - option will let you limit the number of items in the result
			--offset - used as a start index from nth item to retreive`,
	Run: handlers.TableHandle(),
}

func init() {
	listCmd.AddCommand(tableCmd) // adds child to table command whose parent is list.	
	tableCmd.PersistentFlags().StringVarP(&namespace, "schema", "s", config.Data.DefaultNameSpace, "Schema name")

}
```

So the command flags blah blah is already you can try running `go run main.go list table`. This will invoke whatever is assigned to the Run property. In our case I have seperated the handling of business function to handlers. Also note I hae tried using `config.Data.DefaultNameSpace` config using a struct so that we make no mistakes when getting config by string values.

This is how the config class looks

```go
var (
	// Data ...
	Data configuration
)

//Configuration ...
type configuration struct {
	ConnectionString string
	DefaultNameSpace string
}

func init() {
	viper.AddConfigPath(".")
	viper.SetConfigType("yaml")
	viper.SetConfigName(".config")

	viper.AutomaticEnv()

	if err := viper.ReadInConfig(); err == nil {
		err := viper.Unmarshal(&Data)
		if err != nil {
			fmt.Printf("Unable to decode into struct, %v", err)
		}
	}
}
```

If you observe there is a method called [init](https://golang.org/doc/effective_go.html#init) this method is called as soon it is tried to access. Here is a nice explaination from [stackoverflow answer](https://stackoverflow.com/a/49831018/713149).


```yaml
#.config.yaml

ConnectionString: user=bitsmonkey password=password host=db.com port=5432 dbname=jimmy connect_timeout=20 sslmode=verify-full
PGSSLROOTCERT: cert.pem
DefaultNameSpace: ns
```

This line `tableCmd.PersistentFlags` basically says I need a flag that will be used across all my children commands in this case.

If you look at the main its clean, all it does is calls the root command `cmd.Execute()`.

```go
//TableHandle ...
func TableHandle() func(cmd *cobra.Command, args []string) {

	return func(cmd *cobra.Command, args []string) {
		namespace, err := cmd.Flags().GetString("schema")

		if err != nil {
			log.Println(err)
			return
		}
		log.Println("Namespace : ", namespace)
		
		limit, err := cmd.Flags().GetInt("limit")
		if err != nil {
			log.Println(err)
			return
		}
		log.Println("Limit: ", limit)

		offset, err := cmd.Flags().GetInt("offset")
		if err != nil {
			log.Println(err)
			return
		}
		log.Println("Offset: ", offset)

		ddlRepo := repo.Ddl{}
		if result, err := ddlRepo.GetAllTablesIn(namespace, limit, offset); err != nil {
			log.Println("Error when retreiving tables in namespace", namespace)
		} else {
			fmt.Println("---------List of Tables---------\n", strings.Join(result, "\n"))
		}

	}
}
```

This is should be a good start for writing any cli app anymore. Code on!

Photo by [Pankaj Patel](https://unsplash.com/@pankajpatel?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText)
