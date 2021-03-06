﻿---
layout: default
title: Getting Started
type: about
---

# Getting Started

## about_GettingStarted

# SHORT DESCRIPTION

Here is a super simple Polaris application to get you started

```ps
Import-Module Polaris
New-PolarisGetRoute -Path "/" -Scriptblock {
    $Response.Send('Hello World!')
}

Start-Polaris
```

The application starts a server listening on port 8080 (the default port) for connections. It will:

- Respond with "Hello World!" to requests to the root url (/)
- Respond with a 404 not found to any other route (ex. `/DoesNotExist`)

I can get a response from the server by either opening a browser to <http://localhost:8080/> or running the following PowerShell command:

Command:

```ps
PS> Invoke-RestMethod -Method GET -Uri http://localhost:8080/
```

Output:

```None
Hello World!

PS>
```

# LONG DESCRIPTION

Here's something a bit more realistic for a super basic web application. Let's create two simple pages. One that displays running processes and one that displays running services.

For the first one our route to the page will be "/Services" and we'll just want to display in a simple HTML table the Name, Display Name, and Status of all the services installed on my current machine.

We'll create the Polaris route like this

```ps
New-PolarisGetRoute -Path "/Services" -Scriptblock {
   $RunningServices = Get-Service | Select-Object Name,DisplayName,Status | ConvertTo-Html -Title "Services" | Out-String
   $Response.SetContentType("text/html")
   $Response.Send($RunningServices)
}
```

We're leveraging PowerShell's built in ConvertTo-HTML command to generate a little simple HTML for us from the Get-Process command. We are then setting the content type on the response to **text/html** which tells the browser to render the text response as HTML instead of treating it like a regular text file. We're then sending the HTML generated by the Powershell command on.

The second one we'll want to follow suit and create a route for the page called "/Processes" and display the Process Name, CPU, and the Id properties of the process.

```ps
New-PolarisGetRoute -Path "/Processes" -Scriptblock {
   $RunningProcesses = Get-Process | select ProcessName,CPU,Id | ConvertTo-Html -Title "Processes" | Out-String
   $Response.SetContentType("text/html")
   $Response.Send($RunningProcesses)
}
```

We're doing the same thing here as we are in the first just leveraging the Get-Process command instead. Now we should be able to open a web browser to either <http://localhost:8080/Services> or <http://localhost:8080/Processes> and see a little HTML page with some useful information about our system.

## Basic Routing

Routing refers to determining how an application responds to a client request to a particular endpoint, which is a URI (or path) and a specific HTTP request method (GET, POST, and so on).

You can create a routes in Polaris using either `New-PolarisRoute` or one of the aliases `New-PolarisGetRoute`, `New-PolarisPostRoute`, etc.

Let's show you some basic routes here.

Respond 'Hello World!' on the homepage or root:

```ps
New-PolarisRoute -Method GET -Path "/" -ScriptBlock {
    $Response.Send('Hello World!')
}
```

New POST route at the root of the application:

```ps
New-PolarisRoute -Method POST -Path "/" -ScriptBlock {
    $Respond.Send('Received POST request')
}
```

New PUT route to the /Process route:

```ps
New-PolarisRoute -Method PUT -Path "/Process" -ScriptBlock {
    $Response.Send('Received a PUT request at the /Process route')
}
```

New DELETE route at the /Process route:

```ps
New-PolarisRoute -Method DELETE -Path "/Process" -ScriptBlock {
    $Response.Send('Received a DELETE request at the /Process route')
}
```

See about_Routing for more details on advanced routing

## $Request and $Response

Polaris will create two automatic variables for you that are available inside the scriptblock for each route.

\$Request represents information regarding the incoming request from the client and will have details like the URL requested, the body of the request, the query string, the parameters, etc.

\$Response represents the response that will be sent to the client. You can set response headers, send strings or streams back to the client.

## Static Files

To serve static files for your site such as javascript, images, css and other assets you can use a static file route. Polaris takes the contents of an entire folder and moves them to a temporary PSDrive to prevent directory traversal and then serves any files in that folder automatically detecting the MIME type for the file to send with the response.

Let's say you have a simple static website that you want to serve using Polaris. It's made up of the following files in a folder on the C drive called MyAwesomeSite. The files are as follows:

1. **Index.html** - The home page
2. **Scripts.js** - A few handy javascript functions
3. **MyStyles.css** - Some css to make my site stand out
4. **favicon.ico** - Because every good site nees an icon

I can serve all of these files automatically using the following command:

```ps
New-PolarisStaticRoute -RoutePath "/" -FolderPath "C:\MyAwesomeSite"
```

Now I can just call `Start-Polaris` and the following requests will work from a browser:

```None
http://localhost:8080/index.html
http://localhost:8080/scripts.js
http://localhost:8080/mystyles.css
http://localhost:8080/favicon.ico
```

Polaris also has a built-in directory browser you can enable that can be used to set up a simple file server:

```ps
New-PolarisStaticRoute -RoutePath "/" -FolderPath "C:\MyFolderOfFiles" -EnableDirectoryBrowser $True
```

## Simple APIs

APIs are all the rage today and working with them from a browser means you are going to want JSON. PowerShell provides a serious help to you natively with the `ConvertTo-Json` command and Polaris provides a nice helper method to specify that the string you are sending to the client is JSON.

Let's create a quick API for showing information about running processes:

```ps
New-PolarisRoute -Method GET -ScriptBlock {
    $ProcessInfo = Get-Process | Select ProcessName,Id,CPU | ConvertTo-Json
    $Response.json($ProcessInfo)
}
```

Open up a browser to <http://localhost:8080> and you should see your data in a json response. You can now fetch the data via an HTML page using things like [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
