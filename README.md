# clj-test-containers

[![javahippie](https://circleci.com/gh/javahippie/clj-test-containers.svg?style=svg)](<LINK>)

[![Clojars Project](http://clojars.org/clj-test-containers/latest-version.svg)](http://clojars.org/clj-test-containers)

## What it is
This application is supposed to be a lightweight wrapper around the Testcontainers Java library. 

## What it isn't
This library does not provide tools to include testcontainers in your testing lifecycle. As there are many different test tools with different approaches to testing in the clojure world, handling the lifecycle is up to you.


## Usage

The library provides a set of functions to interact with the testcontainers. A simple exampe could look like this:

```clojure
(require '[clj-test-containers.core :as tc])

(def container (-> (tc/create {:image-name "postgres:12.1"
                               :exposed-ports [5432]
                               :env-vars {"POSTGRES_PASSWORD" "verysecret"}})
                   (tc/bind-filesystem! {:host-path "/tmp"
                                         :container-path "/opt"
                                         :mode :read-only})
                   (tc/start!))

(do-database-testing (:host container)
                     (get (:mapped-ports container) 5432))

(tc/stop! container)
```
## Functions and Properties

### create
Creates a testcontainers instance and returns them 

#### Config parameters:

| Key      	| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| `:image-name`    	| String, mandatory				| The name and label of an image, e.g. `postgres:12.2` |
| `:exposed-ports` 	| Vector with ints, mandatory  | All ports which should be exposed and mapped to a local port |
| `:env-vars` 		| Map      			| A map with environment variables|
| `:command`  		| Vector with strings     | Environment Variables to be set in the container|

#### Result: 

| Key      	| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| `:container`    	| `org.testcontainers.containers.Container` 				| The Testcontainers instance, accessible for everything this library doesn't provide (yet) |
| `:exposed-ports` 	| Vector with ints  | Value of the same input parameter |
| `:env-vars` 		| Map      			| Value of the same input parameter|
| `:host` 	 		| String     | The host for the Docker Container|

#### Example:

```clojure
(create {:image-name "alpine:3.2"
         :exposed-ports [80]
         :env-vars {"MAGIC_NUMBER" "42"
         :command ["/bin/sh" 
                   "-c" 
                   "while true; do echo \"$MAGIC_NUMBER\" | nc -l -p 80; done"]})
```

---

### start!
Starts the Testcontainer, which was defined by `create`

#### Config parameters:

| Key      	| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| First parameter: | | |
| `container-config`| Map, mandatory | Return value of the `create` function |

#### Result: 
| Key      	| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| `:container`    	| `org.testcontainers.containers.Container` 				| The Testcontainers instance, accessible for everything this library doesn't provide (yet) |
| `:exposed-ports` 	| Vector with ints  | Value of the same input parameter |
| `:env-vars` 		| Map      			| Value of the same input parameter|
| `:host` 	 		| String     | The host for the Docker Container|
| `:id` 				| String | The ID of the started docker container|
| `:mapped-ports` 	 		| Map     | A map containing the container port as key and the mapped local port as a value|

#### Example:

```clojure
(def container (create {:image-name "alpine:3.2"
                        :exposed-ports [80]
                        :env-vars {"MAGIC_NUMBER" "42"})
		            
(start! container)    
```

---


### stop!
Stops the Testcontainer, which was defined by `create`

#### Config parameters:

| Key      	| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| First parameter: | | |
| `container-config`| Map, mandatory | Return value of the `create` function |

#### Result: 
The `container-config`

#### Example:

```clojure
(def container (create {:image-name "alpine:3.2"
                        :exposed-ports [80]
                        :env-vars {"MAGIC_NUMBER" "42"})
                        
(start! container)    
(stop! container)    
```

---


### map-classpath-resource!
Maps a resource from your classpath into the containers file system

#### Config parameters:

| Key      	| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| First parameter: | | |
| `container-config`| Map, mandatory | Return value of the `create` function |
| Second parameter: | | |
| `:resource-path`    	| String, mandatory 				| Path of your classpath resource |
| `:container-path` 	| String, mandatory | Path, to which the resource should be mapped |
| `:mode` 		| Keyword, mandatory      			| `:read-only` or `:read-write` |


#### Result: 
The `container-config`

#### Example:

```clojure
(map-classpath-resource! container {:resource-path "test.sql"
                             	      :container-path "/opt/test.sql"
                                    :mode :read-only})
```

---


### bind-filesystem!
Binds a path from your local filesystem into the Docker container as a volume

#### Config parameters:

| Key      	| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| First parameter: | | |
| `container-config`| Map, mandatory | Return value of the `create` function |
| Second parameter: | | |
| `:host-path`    	| String , mandatory				| Path on your local filesystem |
| `:container-path` 	| String, mandatory | Path, to which the resource should be mapped |
| `:mode` 		| Keyword, mandatory      			| `:read-only` or `:read-write` |


#### Result: 
The `container-config`

#### Example:

```clojure
(bind-filesystem! container {:host-path "."
                             :container-path "/opt"
                             :mode :read-only})
```

---


### copy-file-to-container!
Copies a file from your filesystem or classpath into the running container

#### Config parameters:

| Key      	| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| First parameter: | | |
| `container-config`| Map, mandatory | Return value of the `create` function |
| Second parameter: | | |
| `:path` 			| String, mandatory | Path to a classpath resource *or* file on your filesystem |
| `:host-path`    	| String, mandatory 				| Path, to which the file should be copied |
| `:type` 			| Keyword, mandatory      			| `:classpath-resource` or `:host-path` |


#### Result: 
The `container-config`

#### Example:

```clojure
(copy-file-to-container! container {:path "test.sql"
                                    :container-path "/opt/test.sql"
                                    :type :host-path})
```

---


### execute-command!
Executes a command in the running container, and returns the result

#### Config parameters:

| Key      	| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| First parameter: | | |
| `container-config`| Map, mandatory | Return value of the `create` function |
| Second parameter: | | |
| `command` 			| Vector with Strings, mandatory | A vector containing the command and its parameters |



#### Result: 
| Key      			| Type          		| Description  |
| ------------- 		|:-------------		| :-----|
| `:exit-code`		| int 	 				| Exit code of the executed command |
| `:stdout`			| String 				| Content of stdout |
| `:stdin`			| String 				| Content of stdin |

#### Example:

```clojure
(execute-command! container ["tail" "/opt/test.sql"])
```



## License

Copyright © 2020 Tim Zöller

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.
