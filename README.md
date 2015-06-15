<!--
title: Algernon
description: Web server with built-in support for Lua, Markdown, Amber, GCSS, JSX, Bolt, Tollbooth, Pie, Graceful, Permissions2, users and permissions
keywords: http2, HTTP/2, web server, http, go, golang, github, algernon, lua, markdown, amber, GCSS, JSX, permissions2, React, Bolt, Three.js, graceful, pie, tollbooth
-->

<a href="https://github.com/xyproto/algernon"><img src="https://raw.github.com/xyproto/algernon/master/img/algernon_logo4.png" style="margin-left: 2em"></a>

Web server with built-in support for HTTP/2, Lua, Markdown, Amber, GCSS, JSX, Bolt, rate limiting, graceful shutdown, plugins, users and permissions.

[![Build Status](https://travis-ci.org/xyproto/algernon.svg?branch=master)](https://travis-ci.org/xyproto/algernon) [![GoDoc](https://godoc.org/github.com/xyproto/algernon?status.svg)](http://godoc.org/github.com/xyproto/algernon) [![Gitter](https://img.shields.io/badge/gitter-chat-green.svg?style=flat)](https://gitter.im/xyproto/algernon?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=body_badge) [![license](http://img.shields.io/badge/license-MIT-red.svg?style=flat)](https://raw.githubusercontent.com/xyproto/algernon/master/LICENSE)

Technologies
------------

Written in [Go](https://golang.org). Uses [Bolt](https://github.com/boltdb/bolt) or [Redis](http://redis.io) as the database backend, [permissions2](https://github.com/xyproto/permissions2) for handling users and permissions, [gopher-lua](https://github.com/yuin/gopher-lua) for interpreting and running Lua, [http2](https://github.com/bradfitz/http2) for serving HTTP/2, [blackfriday](https://github.com/russross/blackfriday) for Markdown rendering, [amber](https://github.com/eknkc/amber) for Amber templates and [GCSS](https://github.com/yosssi/gcss) for CSS preprocessing. [logrus](https://github.com/Sirupsen/logrus) is used for logging, [risotto](https://github.com/mamaar/risotto) for converting from JSX to JavaScript, [tollbooth](https://github.com/didip/tollbooth) for rate limiting, [pie](https://github.com/natefinch/pie) for plugins and [graceful](https://github.com/tylerb/graceful) for graceful shutdowns.

[http2check](https://github.com/xyproto/http2check) can be used to confirm that the server is in fact serving [HTTP/2](https://tools.ietf.org/html/rfc7540).


Design decisions
----------------

* HTTP/2 over SSL/TLS (https) is used by default, if a certificate and key is given.
  * If not, regular HTTP is used.
* /data and /repos have user permissions, /admin has admin permissions and / is public, by default. This is configurable.
* The following filenames are special, in prioritized order:
    * index.lua is interpreted as a handler function for the current directory.
    * index.md is rendered as HTML.
    * index.html is outputted as it is, with the correct Content-Type.
    * index.txt is outputted as it is, with the correct Content-Type.
    * index.amber is rendered as HTML.
    * data.lua is interpreted as Lua code, where the functions and variables are made available for Amber and Markdown pages in the same directory.
    * style.gcss is used as the style for Amber and Markdown pages in the same directory.
* The following filename extensions are handled by Algernon:
    * .md is interpreted as Markdown and rendered as a HTML page.
    * .amber is interpreted as Amber and rendered as a HTML page.
    * .gcss is interpreted as GCSS and rendered as a CSS file.
    * .jsx is interpreted as JSX and rendered as a JavaScript file.
    * .lua is interpreted as a Lua script that provides its own output and content type.
* Other files are given a mimetype based on the extension.
* Directories without an index file are shown as a directory listing, where the design is hardcoded.
* UTF-8 is used whenever possible.
* The server can be configured by commandline flags or with a lua script, but no configuration should be needed for getting started.

Features and limitations
------------------------

* Supports HTTP/2, with or without HTTPS.
* Also supports regular HTTP.
* Can use Lua scripts as handlers for HTTP requests.
* Works on Linux, OS X and 64-bit Windows.
* Algernon is compiled to native. It's reasonably fast.
* The [Lua interpreter](https://github.com/yuin/gopher-lua) is compiled into the executable.
* The use of Lua allows for short development cycles, where code is interpreted when the page is refreshed (there is an auto-refresh feature).
* Self-contained Algernon applications can be zipped into an archive (ending with `.zip` or `.alg`) and be loaded at start.
* Built-in support for [Markdown](https://github.com/russross/blackfriday), [Amber](https://github.com/eknkc/amber), [GCSS](https://github.com/yosssi/gcss) and [JSX](https://github.com/mamaar/risotto).
* Redis is used for the database backend, by default.
* Algernon will fall back to the built-in Bolt database if no Redis server is available.
* The HTML title for a rendered Markdown page can be provided by the first line specifying the title, like this: `title: Title goes here`. This is a subset of MultiMarkdown.
* No file converters needs to run in the background (like for SASS). Files are converted on the fly.
* If `-autorefresh` is enabled, the browser will automatically refresh pages when the source files are changed. Works for Markdown, Lua error pages and Amber (including GCSS and *data.lua*). This only works on Linux and OS X, for now. If listening for changes on too many files, the OS limit for the number of open files may be reached.
* Includes an interactive REPL.
* If only given a Markdown filename as the first argument, it will be served on port 3000, without using any database, as regular HTTP. Handy for viewing `README.md` files locally.
* Full multithreading. All available CPUs will be used.
* Supports rate limiting, by using [tollbooth](https://github.com/didip/tollbooth).
* The `help` command is available at the Lua REPL, for a quick overview of the available Lua functions.
* Can load plugins written in any language. Plugins must offer the `Lua.Code` and `Lua.Help` functions and talk JSON-RPC over stderr+stdin. See [pie](https://github.com/natefinch/pie) for more information. Sample plugins for Go and Python are in the `plugins` directory.
* Thread-safe file caching is built-in, with several available cache modes (for only caching images, for example).


Overview
--------

Running Algernon (screenshot from an earlier version):

<img src="https://raw.github.com/xyproto/algernon/master/img/algernon_redis_054.png">

---

ASCII diagram:

```text
+----------------------------------+-----------------------------------+
|                                  |                                   |
|   Web pages                      |   Styling                         |
|                                  |                                   |
|   Amber instead of HTML:         |   GCSS instead of CSS:            |
|   * Easier to read and write.    |   * Easier to read and write.     |
|   * Easy to add structure.       |   * Easy to add more structure.   |
|   * Can refresh when saving.     |   * Less repetition. DRY.         |
|                                  |                                   |
+----------------------------------+-----------------------------------+
|                                  |                                   |
|   Server side                    |   JavaScript                      |
|                                  |                                   |
|   Lua for providing data:        |   JSX instead of JavaScript:      |
|   * Uses the database backend.   |   * Can build a virtual DOM.      |
|   * Can easily provide data to   |   * Use together with React for   |
|     Amber templates.             |     building single-page apps.    |
|                                  |                                   |
+----------------------------------+-----------------------------------+
|                                  |                                   |
|   Markdown                       |   Database backends               |
|                                  |                                   |
|   For static pages:              |   Redis or Bolt for the backend:  |
|   * Easy content creation.       |   * Both are fast.                |
|   * Easy to style with GCSS.     |   * Bolt is included.             |
|   * Can refresh when saving.     |   * Redis scales easily.          |
|                                  |                                   |
+----------------------------------+-----------------------------------+
```

* Redis offers good [data persistence](http://redis.io/topics/persistence).
* Bolt is a [pure key/value store](https://github.com/boltdb/bolt), written in Go.

Screenshots
-----------

<img src="https://raw.github.com/xyproto/algernon/master/img/algernon_markdown.png">

*Markdown can easily be styled with GCSS.*

---

<img src="https://raw.github.com/xyproto/algernon/master/img/algernon_lua_error.png">

*This is how errors in Lua scripts are handled, when Debug mode is enabled.*

---

<img src="https://raw.github.com/xyproto/algernon/master/img/algernon_threejs.png">

*One of the poems of Algernon Charles Swinburne, with three rotating tori in the background.*
*Uses CSS3 for the gaussian blur and [three.js](http://threejs.org) for the 3D graphics.*

---

<img src="https://raw.github.com/xyproto/algernon/master/img/prettify.png">

*Screenshot of the <strong>prettify</strong> sample. Served from a single Lua script.*

---

<img src="https://raw.github.com/xyproto/algernon/master/img/algernon_react.png">

*JSX transforms are built-in. Using [React](https://facebook.github.io/react/) together with Algernon is easy.*


Installation instructions
-------------------------

##### OS X, latest release, built from source:
  * Install [Homebrew](https://brew.sh):
    `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
  * Install the latest version of Algernon:
  `brew install https://raw.githubusercontent.com/xyproto/algernon/master/system/homebrew/algernon.rb`

##### Arch Linux, latest release, built from source:
  * Install `algernon` from AUR, using your favorite AUR helper

##### Any system where go has been installed (and $GOPATH has been set), from source (master branch)
  * `go get github.com/xyproto/algernon`
  * Optionally, add `$GOPATH/bin` to the PATH


Getting started
---------------

##### Enable HTTP/2 in the browser

* Chrome: go to `chrome://flags/#enable-spdy4`, enable, save and restart the browser.
* Firefox: go to `about:config`, set `network.http.spdy.enabled.http2draft` to `true`. You might need the nightly version of Firefox.

##### Configure the required ports for local use

* You may need to change the firewall settings for port 3000, if you wish to use the default port for exploring the samples.
* For the auto-refresh feature to work, port 5553 must be available (or another host/port of your choosing, if configured otherwise).

##### Prepare for running the samples

* `cd $GOPATH/src/github.com/xyproto/algernon`
* `go build`

##### The "bob" sample, over https

* Run `./bob.sh` to start serving the "bob" sample.
* Visit `https://localhost:3000/` (*note*: it's `https`)

##### All the samples, over http, with auto-refresh enabled

* Run `./samples.sh` to start serving the sample directory.
* Visit `http://localhost:3000/` (*note*: it's `http`)

##### Create your own Algernon application, for regular HTTP

* `mkdir mypage`
* `cd mypage`
* Create a file named `index.lua`, with the following contents:
  `print("Hello, Algernon")`
* Start `algernon -httponly -autorefresh`.
* Visit `http://localhost:3000/`.
* Edit `index.lua` and refresh the browser to see the new result.
* If there were errors, the page will automatically refresh when `index.lua` is changed.
* Markdown and Amber pages will also refresh automatically, as long as `-autorefresh` is used.

##### Create your own Algernon application, for HTTP/2 + HTTPS

* `mkdir mypage`
* `cd mypage`
* Create a file named `index.lua`, with the following contents:
  `print("Hello, Algernon")`
* Create a self-signed certificate, just for testing:
 * `openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 3000 -nodes`
 * Press return at all the prompts, but enter `localhost` at *Common Name*.
* Start `algernon`.
* Visit `https://localhost:3000/`.
* If you have not imported the certificates into the browser, nor used certificates that are signed by trusted certificate authorities, perform the necessary clicks to confirm that you wish to visit this page.
* Edit `index.lua` and refresh the browser to see the result (or a Lua error message, if the script had a problem).


Basic Lua functions
-------------------

* `version()` return the version string for the server.
* `sleep(number)` sleep the given number of seconds (can be a float).
* `log(...)` log the given strings as information. Takes a variable number of strings.
* `warn(...)` log the given strings as a warning. Takes a variable number of strings.
* `error(...)` log the given strings as an error. Takes a variable number of strings.


Lua functions for handling requests
-----------------------------------

* `content(string)` set the Content-Type for a page.
* `method()` return the requested HTTP method (GET, POST etc).
* `print(...)` output text to the browser/client. Takes a variable number of strings.
* `urlpath()` return the requested URL path.
* `header(string)` return the HTTP header in the request, for a given key, or an empty string.
* `setheader(string, string)` set an HTTP header given a key and a value.
* `body()` return the HTTP body in the request (will only read the body once, since it's streamed).
* `status(number)` set a HTTP status code (like 200 or 404). Must come before other output.
* `error(string, number)` output a message and set a HTTP status code.
* `scriptdir(...)` return the directory where the script is running. If a filename is given, then the path to where the script is running, joined with a path separator and the given filename, is returned.
* `serverdir(...)` return the directory where the server is running. If a filename is given, then the path to where the server is running, joined with a path separator and the given filename, is returned.
* `serve(string)` serve a file that exists in the same directory as the script.
* `formdata()` return a table with keys and values as given in a posted form, or as given in the URL (`/some/page?x=7` makes the key `x` with the value `7` available).

Lua functions for formatted output
----------------------------------

* `mprint(...)` output Markdown to the browser/client. The given text is converted from Markdown to HTML. Takes a variable number of strings.
* `aprint(...)` output Amber to the browser/client. The given text is converted from Amber to HTML. Takes a variable number of strings.
* `gprint(...)` output GCSS to the browser/client. The given text is converted from GCSS to CSS. Takes a variable number of strings.
* `jprint(...)` output JSX to the browser/client. The given text is converted from JSX to JavaScript. Takes a variable number of strings.
* `ToJSON(table)` return a JSON string, given a Lua table with ints or strings.


Lua functions for plugins
-------------------------

* `Plugin(string)` load a plugin given the path to an executable. Returns true if successful. Will return the plugin help text if called on the Lua prompt.
* `PluginCode(string) -> string` returns the Lua code as returned by the Lua.Code function in the plugin, given a plugin path. May return an empty string.
* `CallPlugin(string, string, ...) -> string` takes a plugin path, function name and arguments. Returns an empty string if the function call fails, or the results as a JSON string if successful.


Lua functions for code libraries
--------------------------------

These functions can be used in combination with the plugin functions for storing Lua code returned by plugins when serverconf.lua is loaded, then retrieve the Lua code later, when handling requests.

* `CodeLib() -> userdata` creates a code library object. Optionally takes a data structure name as the first parameter.
* `codelib:add(string, string) -> bool` given a namespace and Lua code, add the given code to the namespace. Returns true if successful.
* `codelib:set(string, string) -> bool` given a namespace and Lua code, set the given code as the only code in the namespace. Returns true if successful.
* `codelib:get(string) -> string` given a namespace, return Lua code or an empty string.
* `codelib:import(string) -> bool` import (eval) code from the given namespace into the current Lua state. Returns true on success.
* `codelib:clear() -> bool` completely clear the code library. Returns true of successful.


Lua functions for the file cache
--------------------------------

* `CacheStats() -> string` return stats about the file cache.
* `ClearCache()` clear the file cache.


Lua functions for data structures
---------------------------------

##### Set

~~~c
// Get or create a database-backed Set (takes a name, returns a set object)
Set(string) -> userdata

// Add an element to the set
set:add(string)

// Remove an element from the set
set:del(string)

// Check if a set contains a value
// Returns true only if the value exists and there were no errors.
set:has(string) -> bool

// Get all members of the set
set:getall() -> table

// Remove the set itself. Returns true if successful.
set:remove() -> bool

// Clear the set
set:clear() -> bool
~~~

##### List

~~~c
// Get or create a database-backed List (takes a name, returns a list object)
List(string) -> userdata

// Add an element to the list
list:add(string)

// Get all members of the list
list:getall() -> table

// Get the last element of the list
// The returned value can be empty
list:getlast() -> string

// Get the N last elements of the list
list:getlastn(number) -> table

// Remove the list itself. Returns true if successful.
list:remove() -> bool

// Clear the list. Returns true if successful.
list:clear() -> bool
~~~

##### HashMap

~~~c
// Get or create a database-backed HashMap (takes a name, returns a hash map object)
HashMap(string) -> userdata

// For a given element id (for instance a user id), set a key
// (for instance "password") and a value.
// Returns true if successful.
hash:set(string, string, string) -> bool

// For a given element id (for instance a user id), and a key
// (for instance "password"), return a value.
// Returns a value only if they key was found and if there were no errors.
hash:get(string, string) -> string

// For a given element id (for instance a user id), and a key
// (for instance "password"), check if the key exists in the hash map.
// Returns true only if it exists and there were no errors.
hash:has(string, string) -> bool

// For a given element id (for instance a user id), check if it exists.
// Returns true only if it exists and there were no errors.
hash:exists(string) -> bool

// Get all keys of the hash map
hash:getall() -> table

// Remove a key for an entry in a hash map
// (for instance the email field for a user)
// Returns true if successful
hash:delkey(string, string) -> bool

// Remove an element (for instance a user)
// Returns true if successful
hash:del(string) -> bool

// Remove the hash map itself. Returns true if successful.
hash:remove() -> bool

// Clear the hash map. Returns true if successful.
hash:clear() -> bool
~~~

##### KeyValue

~~~c
// Get or create a database-backed KeyValue collection (takes a name, returns a key/value object)
KeyValue(string) -> userdata

// Set a key and value. Returns true if successful.
kv:set(string, string) -> bool

// Takes a key, returns a value.
// Returns an empty string if the function fails.
kv:get(string) -> string

// Takes a key, returns the value+1.
// Creates a key/value and returns "1" if it did not already exist.
// Returns an empty string if the function fails.
kv:inc(string) -> string

// Remove a key. Returns true if successful.
kv:del(string) -> bool

// Remove the KeyValue itself. Returns true if successful.
kv:remove() -> bool

// Clear the KeyValue. Returns true if successful.
kv:clear() -> bool
~~~


Lua functions for handling users and permissions
------------------------------------------------

~~~c
// Check if the current user has "user" rights
UserRights() -> bool

// Check if the given username exists
HasUser(string) -> bool

// Get the value from the given boolean field
// Takes a username and fieldname
BooleanField(string, string) -> bool

// Save a value as a boolean field
// Takes a username, fieldname and boolean value
SetBooleanField(string, string, bool)

// Check if a given username is confirmed
IsConfirmed(string) -> bool

// Check if a given username is logged in
IsLoggedIn(string) -> bool

// Check if the current user has "admin rights"
AdminRights() -> bool

// Check if a given username is an admin
IsAdmin(string) -> bool

// Get the username stored in a cookie, or an empty string
UsernameCookie() -> string

// Store the username in a cookie, returns true if successful
SetUsernameCookie(string) -> bool

// Clear the login cookie
ClearCookie()

// Get a table containing all usernames
AllUsernames() -> table

// Get the email for a given username, or an empty string
Email(string) -> string

// Get the password hash for a given username, or an empty string
PasswordHash(string) -> string

// Get all unconfirmed usernames
AllUnconfirmedUsernames() -> table

// Get a confirmation code that can be given to a user,
// or an empty string. Takes a username.
ConfirmationCode(string) -> string

// Add a user to the list of unconfirmed users
// Takes a username and a confirmation code
AddUnconfirmed(string, string)

// Remove a user from the list of unconfirmed users
// Takes a username
RemoveUnconfirmed(string)

// Mark a user as confirmed
// Takes a username
MarkConfirmed(string)

// Removes a user
// Takes a username
RemoveUser(string)

// Make a user an admin
// Takes a username
SetAdminStatus(string)

// Make an admin user a regular user
// Takes a username
RemoveAdminStatus(string)

// Add a user
// Takes a username, password and email
AddUser(string, string, string)

// Set a user as logged in on the server (not cookie)
// Takes a username
SetLoggedIn(string)

// Set a user as logged out on the server (not cookie)
// Takes a username
SetLoggedOut(string)

// Log in a user, both on the server and with a cookie
// Takes a username
Login(string)

// Log in a user, both on the server and with a cookie
// Takes a username. Returns true if the cookie was set successfully.
CookieLogin(string) -> bool

// Log out a user, on the server (which is enough)
// Takes a username
Logout(string)

// Get the current username, from the cookie
Username() -> string

// Get the current cookie timeout
// Takes a username
CookieTimeout(string) -> number

// Set the current cookie timeout
// Takes a timeout number, measured in seconds
SetCookieTimeout(number)

// Get the current password hashing algorithm (bcrypt, bcrypt+ or sha256)
PasswordAlgo() -> string

// Set the current password hashing algorithm (bcrypt, bcrypt+ or sha256)
// Takes a string
SetPasswordAlgo(string)

// Hash the password
// Takes a username and password (username can be used for salting)
HashPassword(string, string) -> string

// Check if a given username and password is correct
// Takes a username and password
CorrectPassword(string, string) -> bool

// Checks if a confirmation code is already in use
// Takes a confirmation code
AlreadyHasConfirmationCode(string) -> bool

// Find a username based on a given confirmation code,
// or returns an empty string. Takes a confirmation code
FindUserByConfirmationCode(string) -> string

// Mark a user as confirmed
// Takes a username
Confirm(string)

// Mark a user as confirmed, returns true if successful
// Takes a confirmation code
ConfirmUserByConfirmationCode(string) -> bool

// Set the minimum confirmation code length
// Takes the minimum number of characters
SetMinimumConfirmationCodeLength(number)

// Generates a unique confirmation code, or an empty string
GenerateUniqueConfirmationCode() -> string
~~~

Lua functions for use when streaming
------------------------------------

* `flush()` sends what has been outputted so far to the client.


Lua functions that are only available for the server configuration file
-----------------------------------------------------------------------

* `SetAddr(string)` set the default address for the server on the form [host][:port].
* `ClearPermissions()` reset the URL prefixes and make everything *public*.
* `AddAdminPrefix(string)` add an URL prefix that will have *admin* rights.
* `AddUserPrefix(string)` add an URL prefix that will have *user* rights.
* `DenyHandler(function)` provide a lua function that will be used as the permission denied handler.
* `ServerInfo() -> string` return a string with various server information.
* `LogTo(string) -> bool` log to the given filename. If the filename is an empty string, log to stderr. Returns true if successful.
* `version() -> string` returns the version string for the server.
* `log(...)` logs the given strings as INFO. Takes a variable number of strings.
* `warn(...)` logs the given strings as WARN. Takes a variable number of strings.
* `OnReady(function)` provide a lua function that will be run once, when the server is ready to start serving.


Releases
--------

* [Arch Linux package](https://aur.archlinux.org/packages/algernon) in the AUR.
* [Windows executable](https://github.com/xyproto/algernon/releases/tag/v0.62-win8-64).
* [OS X homebrew package](https://raw.githubusercontent.com/xyproto/algernon/master/system/homebrew/algernon.rb)
* Source releases are tagged as such.


General information
-------------------

* Version: 0.74
* License: MIT
* Alexander F Rødseth

