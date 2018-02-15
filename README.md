IPFS-Store (service)
======

**IPFS-Store** aims is to provide an easy to use IPFS storage service with search capabilities for your project.

The service is composed of three main components:

- **IPFS Node**
- **ElasticSearch engine** (other implementations coming soon)
- **IPFS-Store** which provides an API to store/index/search/fetch

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. 

### Prerequisites

- Java 8
- Maven 
- Docker (optional)


### Build

1. After checking out the code, navigate to the root directory
```
$ cd /path/to/ipfs-store/
```

2. Compile, test and package the project
```
$ mvn clean package
```

3. Run the project

a. If you have an existing running IPFS node and ElasticSearch 

```
$ docker build  . -t kauri/ipfs-store:latest

$ export IPFS_HOST=localhost
$ export IPFS_PORT=5001
$ export ELASTIC_CLUSTERNODES=localhost:9300
$ export ELASTIC_CLUSTERNAME=`elasticsearch`

$ docker run -p 8040:8040 kauri/ipfs-store
```

b. If you prefer build all-in-one with docker-compose
```
$ docker-compose -f docker-compose.yml build
$ docker-compose -f docker-compose.yml up
```

## API Documentation



### Overview

| Operation | Description | Method | URI |
| -------- | -------- | -------- | -------- |
| Store content | Store content into IPFS |POST | /ipfs-store/store |
| Index content | Index content |POST | /ipfs-store/index |
| Get content | Get content | GET | /ipfs-store/fetch/{index}/{hash} |
| Search content | Search content | POST | /ipfs-store/search/{index} |

### Details

#### Store content

Store a content (any type) in IPFS 

-   **URL:** `/ipfs-store/store`    
-   **Method:** `POST`
-   **Header:** `N/A`
-   **URL Params:** `N/A`
-   **Data Params:** `file: [content]`

-   **Sample Request:**
```
$ curl -X POST \
    'http://localhost:8040/ipfs-store/store' \
    -F 'file=@/home/gjeanmart/hello.pdf'
```

-   **Success Response:**
    -   **Code:** 200  
        **Content:** 
```
{
    "hash": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o"
}
```

---------------------------

#### Index content

Index IPFS content into the search engine

-   **URL** `/ipfs-store/index`
-   **Method:** `POST`
-   **Header:**  

| Key | Value | 
| -------- | -------- |
| content-type | application/json |


-   **URL Params** `N/A`
-   **Data Params**



```
{
	"index": "documents", 
	"id": "hello_doc",
	"content_type": "application/pdf",
	"hash": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o",
	"index_fields": [
		{
			"name": "title",
			"value": "Hello Doc"
		}, 
		{
			"name": "author",
			"value": "Gregoire Jeanmart"
		}, 
		{
			"name": "votes",
			"value": 10
		}, 
		{
			"name": "date_created",
			"value": 1518700549
		}
	]
}
```
   
-   **Sample Request:**
    
```
curl -X POST \
    'http://localhost:8040/ipfs-store/index' \
    -H 'content-type: application/json' \  
    -d '{"index":"documents","id":"hello_doc","content_type":"application/pdf","hash":"QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o","index_fields":[{"name":"title","value":"Hello Doc"},{"name":"author","value":"Gregoire Jeanmart"},{"name":"votes","value":10},{"name":"date_created","value":1518700549}]}'
``` 
   
-   **Success Response:**
    
    -   **Code:** 200  
        **Content:** 
```
{
    "index": "documents",
    "id": "hello_doc",
    "hash": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o"
}
```

---------------------------

#### Get content

Get content

-   **URL** `http://localhost:8040/ipfs-store/fetch/{index}/{hash}`
-   **Method:** `GET`
-   **Header:**  `N/A`
-   **URL Params** `N/A`
    
-   **Sample Request:**
    
```
$ curl \
    'http://localhost:8040/ipfs-store/fetch/documents/QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o' \
    -o hello_doc.pdf 
``` 
    
-   **Success Response:**
    
    -   **Code:** 200  
        **Content:** (file)

---------------------------

#### Search contents

Search content accross an index using a dedicated query language

-   **URL** `http://localhost:8040/ipfs-store/search/{index}`
-   **Method:** `POST`
-   **Header:**  

| Key | Value | 
| -------- | -------- |
| content-type | application/json |

-   **URL Params** 

| Name | Type | Mandatory | Default | Description |
| -------- | -------- | -------- | -------- | -------- |
| pageNo | Int | no | 1 | Page Number |
| pageSize | Int | no | 20 | Page Size / Limit |
| sort | String | no |  | Sorting attribute |
| dir | ASC/DESC | no | ASC | Sorting direction |


-   **Data Params** 

The `search` operation allows to run a multi-criteria search against an index. The body combines a list of filters : 

| Name | Type | Description |
| -------- | -------- | -------- | 
| name | String | Index field for the search | 
| operation | see below | Operation to run against the index field | 
| value | String | Value to compare withg |



| Operation | Description |
| -------- | -------- |
| equals | Equals |
| not_equals | Not equals | 
| contains | Contains the word/phrase | 
| in | in the following list | 
| gt | Greater than | 
| gte | Greater than or Equals | 
| lt | Less than  | 
| lte | Less than or Equals | 


```
{
	"query": [
		{
			"name": "title",
			"operation": "contains",
			"value": "Hello"
		},
		{
			"name": "author",
			"operation": "equals",
			"value": "Gregoire Jeanmart"
		},
		{
			"name": "votes",
			"operation": "lt",
			"value": "5"
		}
	]
}
```



-   **Success Response:**
    
    -   **Code:** 200  
        **Content:** 
        
```
{
  "content": [
    {
      "index": "documents",
      "id": "hello_doc",
      "hash": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o",
      "content_type": "application/pdf",
      "index_fields": [
        {
          "name": "__content_type",
          "value": "application/pdf"
        },
        {
          "name": "__hash",
          "value": "QmWPCRv8jBfr9sDjKuB5sxpVzXhMycZzwqxifrZZdQ6K9o"
        },
        {
          "name": "title",
          "value": "Hello Doc"
        },
        {
          "name": "author",
          "value": "Gregoire Jeanmart"
        },
        {
          "name": "votes",
          "value": 10
        },
        {
          "name": "date_created",
          "value": 1518700549
        }
      ]
    }
  ]
}
],
"sort": null,
"firstPage": false,
"totalElements": 4,
"lastPage": true,
"totalPages": 1,
"numberOfElements": 4,
"size": 20,
"number": 1
}
```



## Clients

### Java


### Spring


### Javascript




## TODO

- Implements clients
- Full text search
- 

