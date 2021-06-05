# MarkLogic REST API

MarkLogic provides a REST API which lets us interact with content on a database.

Let's figure out how it works.

## Creating a REST API

First we need to create a REST API for our database.

Create a config.xml file with these REST API parameters:

```xml
<rest-api xmlns="http://marklogic.com/rest-api">
  <name>RESTAPI</name>
  <database>Documents</database>
  <port>8020</port>
</rest-api>
```

POST the config to MarkLogic as follows:

```text
curl -X POST --anyauth --user admin:admin -d @"./config.xml" -H "Content-type: application/xml" http://localhost:8002/LATEST/rest-apis
```

You can get a report of the REST API appservers using this URI: http://localhost:8002/LATEST/rest-apis

## CRUD operations

### Create 

Create a document using HTTP PUT.

```text
curl --anyauth --user admin:admin -X PUT -T ./a.xml -H "Content-type: application/xml" "http://localhost:8020/LATEST/documents?uri=/marc/a"
```

### Setting collections

We can set various pieces of document metadata, including collections, by adding collection parameters.

```text
curl --anyauth --user admin:admin -X PUT -T ./a.xml -H "Content-type: application/xml" "http://localhost:8020/LATEST/documents?uri=/marc/a&collection=foo&collection=bar"
```

Another way to set the metadata is post it.  Create a metadata.xml file:

```xml
<rapi:metadata xmlns:rapi="http://marklogic.com/rest-api"
               xmlns:prop="http://marklogic.com/xdmp/property">
  <rapi:collections>
    <rapi:collection>marccollection</rapi:collection>
  </rapi:collections>
</rapi:metadata>
```

Then post it with this command:

```text
curl --verbose --anyauth --user admin:admin -X PUT -T ./metadata.xml -H "Content-type: application/xml" "http://localhost:8020/LATEST/documents?uri=/marc/a&category=collections"
```

MarkLogic also provides a way to post both the content and metadata at the same time, using a multipart post.


### Read 

* Get a document by URI: http://localhost:8020/LATEST/documents?uri=/marc/a
* Get document collections: http://localhost:8020/LATEST/documents?uri=/marc/a&category=collections
* Get all document metadata: http://localhost:8020/LATEST/documents?uri=/marc/a&category=metadata


### Update 

One way to change the document is simply replace it using the HTTP PUT  again.

MarkLogic also supports a method of patching documents using HTTP POST or HTTP PATCH to the document URI, which POSTS in an xml intruction that explains how to manipulate the document.  The posted content for that scenario looks along these lines:

```xml
<rapi:patch xmlns:rapi="http://marklogic.com/rest-api">
  <rapi:insert context="/inventory/history" position="last-child">
    <modified>2012-11-5</modified>
  </rapi:insert>
  <rapi:delete select="saleExpirationDate"/>
  <rapi:replace select="price" apply="ml.multiply">1.1</rapi:replace>
</rapi:patch>
```

This is explained fully in the official docs: https://docs.marklogic.com/guide/rest-dev/documents

### Delete 

Delete the document with HTTP DELETE.

```text
curl --anyauth --user admin:admin -i -X DELETE http://localhost:8020/LATEST/documents?uri=/marc/a
```

## Search operations

### Return all documents in a collection, or in a directory

This return all documents in a particular collection.  The documents are returned as separate multipart attachments.

```text
curl --anyauth --user admin:admin  -H "Accept: multipart/mixed; boundary=BOUNDARY" "http://localhost:8020/LATEST/search?collection=foo&category=content"
```

It can do paging using start and pageLength pagination:

```text
curl --anyauth --user admin:admin  -H "Accept: multipart/mixed; boundary=BOUNDARY" "http://localhost:8020/LATEST/search?collection=foo&category=content&start=1&pageLength=2"
```

### Extracting nodes and returning a single XML response

We can get a single XML response using the extract-document-data feature.

We have to construct and post a search query:

* collection-query is searching for documents in collection foo
* extract-path specifies an XPath in the matched documents to return.

search.xml
```xml
<search xmlns="http://marklogic.com/appservices/search">
  <query>
    <collection-query>
      <uri>foo</uri>
    </collection-query>
  </query>
  <options xmlns="http://marklogic.com/appservices/search">
    <extract-document-data selected="include">
      <extract-path>/</extract-path>
    </extract-document-data>
    <return-results>true</return-results>
  </options>
</search>
```

Here's the curl request that posts the query:

```text
curl --anyauth --user admin:admin -H "Content-Type: application/xml" -X POST -d @./search.xml "http://localhost:8020/LATEST/search"
```

Here's the result.  The "document" nodes are the XML documents from my database.

```xml
<search:response snippet-format="snippet" total="3" start="1" page-length="10" selected="include" xmlns:search="http://marklogic.com/appservices/search">
  <search:result index="1" uri="/marc/a" path="fn:doc(&quot;/marc/a&quot;)" score="0" confidence="0" fitness="0" href="/v1/documents?uri=%2Fmarc%2Fa" mimetype="application/xml" format="xml">
    <search:snippet>
      <search:match path="fn:doc(&quot;/marc/a&quot;)/document">document a hello this is document a has been modified</search:match>
    </search:snippet>
    <search:extracted kind="document">
        <document>
            <name>document a</name>
            <text>hello this is document a</text>
            <it>has been modified</it>
        </document>
    </search:extracted>
  </search:result>
  <search:result index="2" uri="/marc/b" path="fn:doc(&quot;/marc/b&quot;)" score="0" confidence="0" fitness="0" href="/v1/documents?uri=%2Fmarc%2Fb" mimetype="application/xml" format="xml">
    <search:snippet>
      <search:match path="fn:doc(&quot;/marc/b&quot;)/document">document a hello this is document a has been modified</search:match>
    </search:snippet>
    <search:extracted kind="document">
        <document>
            <name>document a</name>
            <text>hello this is document a</text>
            <it>has been modified</it>
        </document>
    </search:extracted>
  </search:result>
  <search:result index="3" uri="/marc/c" path="fn:doc(&quot;/marc/c&quot;)" score="0" confidence="0" fitness="0" href="/v1/documents?uri=%2Fmarc%2Fc" mimetype="application/xml" format="xml">
    <search:snippet>
      <search:match path="fn:doc(&quot;/marc/c&quot;)/document">document a hello this is document a has been modified</search:match>
    </search:snippet>
    <search:extracted kind="document">
        <document>
            <name>document a</name>
            <text>hello this is document a</text>
            <it>has been modified</it>
        </document>
    </search:extracted>
  </search:result>
  <search:metrics>
    <search:query-resolution-time>PT0.002372S</search:query-resolution-time>
    <search:snippet-resolution-time>PT0.00182S</search:snippet-resolution-time>
    <search:extract-resolution-time>PT0.001776S</search:extract-resolution-time>
    <search:total-time>PT0.009493S</search:total-time>
  </search:metrics>
</search:response>
```

The MarkLogic REST API has quite a lot of features, but it's quite unintuitive.   It requires POSTS of parameters in complex XML structures, and frequently uses multipart requests and responses.  Of course, that is required if the content being transferred was multiple content types.
