# Dealing with DateTime Internationalization on WebSharper

This article describes how to setup a WebSharper client-server application to handle DateTime with internationalization.

We are going to use pt-BR (Brazilian Portuguese) language, but changing it to other language might be straighforward.

## Setting up UICulture/Culture

### Web.config

add the globalization tag to Web.config file:

```
<configuration>
  ...
  <system.web>
    ...
    <globalization culture="pt-BR" uiCulture="pt-BR"/>
  </system.web>
    ...
</configuration>
```

## Scenario 1: creating the date at the server and consuming it through RPC

In this scenario the date is created using the localized culture and WebSharper converts it correctly to Javascript.

Ex.:

client-side:
```
...
open WebSharper.Javascript

[<JavaScript>]
let PageTest () =
    async {
        let! dtServer = Server.CreateDate()
        Console.log (dtServer.ToShortDateString())
        Console.log (dtServer.ToLongTimeString())
        
        return Doc.Empty
    }
    |> Doc.Async


```
> Console.log Output:

> 07/07/2019

> 20:58:31

server-side:
```
open System.Diagnostics

module Server =

    [<Rpc>]
    let CreateDate () =
        async {
            // The date is created using the current culture
            let date = DateTime.Now
            Trace.WriteLine("CreateDate:")
            Trace.WriteLine(date)
            return date
        }
```
> Trace Output:

> CreateDate:

> 07/07/2019 20:58:31

### Scenario 2: creating a date at the client side, sending and getting it back from the server

Creating a date at client side and sending it to server requires the ToLocalTime() function, for proper conversion.


```
...
open WebSharper.Javascript

[<JavaScript>]
let PageTest () =
    async {
        let dtClient = DateTime.Now
        let! dtGetBack = Services.ReturnClientDate dtClient
        let! dtGetBackAsLocal = Services.ReturnClientDateAsLocal dtClient

        Console.log (dtGetBack.ToShortDateString())
        Console.log (dtGetBack.ToLongTimeString())
        
        Console.log (dtGetBackAsLocal.ToShortDateString())
        Console.log (dtGetBackAsLocal.ToLongTimeString())

        return Doc.Empty
    }
    |> Doc.Async


```
Client side date sent and get back from server
> Console.log Output:

> 07/07/2019

> 20:58:31

Client side date sent to server and get back from server - converted to local
> Console.log Output:

> 07/07/2019

> 20:58:31


server-side:
```
open System.Diagnostics

module Server =

    [<Rpc>]
    let ReturnClientDate (date:DateTime) =
        async {
            // The date created at the client is passed in UTC format and is handled
            // correctly by Javascript (at the client side) when returned as is.
            Trace.WriteLine("ReturnClientDate:")
            Trace.WriteLine(date)
            return date
        }

    [<Rpc>]
    let ReturnClientDateAsLocal (date:DateTime) =
        async {
            // The date created at the client is passed in UTC format and is handled
            // correctly by Javascript (at the client side), even when converted to local.
            Trace.WriteLine("ReturnClientDateAsLocal:")
            Trace.WriteLine(date)

            let localDate = date.ToLocalTime()
            Trace.WriteLine(localDate)
            return localDate
        }


```
ReturnClientDate function:
> Trace Output:

> ReturnClientDate:

> 07/07/2019 **23:58:31***

ReturnClientDateAsLocal function:
> Trace Output:

> ReturnClientDateAsLocal:

> 07/07/2019 **23:58:31***

> 07/07/2019 20:58:31
