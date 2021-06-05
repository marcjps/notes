
# MarkLogic Triples

We want a solution that can return both XML and related RDF content at the same time, in a single query.  We may also want to search content using criteria against both XML and RDF.

Here is an experiment in doing that in MarkLogic.

#### Load data

The XQuery code below inserts some XML data and some triples.

I am creating two XML "Person" records, each of which has two corresponding triples.  I have an URI in the XML data which I'm using as an identifier to link the XML and the triples.  Its very rough data for now.  

You can ingest both XML and triples into MarkLogic using many methods.  This is just one way to get some data there.

	xquery version "1.0-ml";

	import module namespace sem = "http://marklogic.com/semantics" 
	  at "/MarkLogic/semantics.xqy";

	xdmp:document-insert(
	  "http://person/marc",
	  <Person>
	    <URI>http://person/marc</URI>
	    <Biography>This is Marc's autobiography content</Biography>    
	  </Person>
	),

	sem:rdf-insert(sem:triple(
	      sem:iri("http://person/marc"),
	      sem:iri("http://data/firstname"),
	      'Marc'), 
	      (), 
	      (),  
	      "mygraph"),

	sem:rdf-insert(sem:triple(
	      sem:iri("http://person/marc"),
	      sem:iri("http://data/surname"),
	      'Sturman'), 
	      (), 
	      (),  
	      "mygraph"),

	xdmp:document-insert(
	  "http://person/john",
	  <Person>
	    <URI>http://person/john</URI>
	    <Biography>This is John's biography content</Biography>    
	  </Person>
	),

	sem:rdf-insert(sem:triple(
	      sem:iri("http://person/john"),
	      sem:iri("http://data/firstname"),
	      'John'), 
	      (), 
	      (),  
	      "mygraph"),

	sem:rdf-insert(sem:triple(
	      sem:iri("http://person/john"),
	      sem:iri("http://data/surname"),
	      'Sheridan'), 
	      (), 
	      (),  
	      "mygraph")

#### Small pieces of the puzzle

Just to show what we have got, in QConsole we can return the XML simply with:

	/Person

Which returns:

	<Person>
		<URI>http://person/john</URI>
		<Biography>This is John's biography content</Biography>
	</Person>
	<Person>
		<URI>http://person/marc</URI>
		<Biography>This is Marc's autobiography content</Biography>
	</Person>

Examine of the database does indeed reveal that the triples in their raw format in MarkLogic are stored as XML.

	<?xml  version="1.0" encoding="UTF-8"?>
	<sem:triples xmlns:sem="http://marklogic.com/semantics">
		<sem:triple>
			<sem:subject>http://person/john</sem:subject>
			<sem:predicate>http://data/firstname</sem:predicate>
			<sem:object datatype="http://www.w3.org/2001/XMLSchema#string">John</sem:object>
		</sem:triple>
	</sem:triples>

However we can run a SPARQL query in QConsole and it does return the triples:
	
	SELECT ?s ?p ?o WHERE { ?s ?p ?o }

Which returns (here as JSON):

	[
		{
			"s":"<http://person/john>",
			"p":"<http://data/firstname>",
			"o":"\"John\""
		},
		{
			"s":"<http://person/marc>",
			"p":"<http://data/firstname>",
			"o":"\"Marc\""
		},
		{
			"s":"<http://person/john>",
			"p":"<http://data/surname>",
			"o":"\"Sheridan\""
		},
		{
			"s":"<http://person/marc>",
			"p":"<http://data/surname>",
			"o":"\"Sturman\""
		}
	]

We can serialise the triples into the usual formats.  Reference: https://docs.marklogic.com/sem:rdf-serialize

#### Loop through XML and return corresponding triples

We want a solution that can return both XML and the related RDF content at the same time.

Since the RDF is stored as XML it would probably be trivial for us to write a XQuery that fetches both the XML and the XML representation of the RDF.  I decided to do an RDF query using SPARQL though, assuming that the semantic team may wish to write SPARQL queries against the triples rather than XQueries.

This code loops through the XML Person elements and then runs a SPARQL query to find matching triples for each XML document.  It returns the data combined together.

	xquery version "1.0-ml";
	import module namespace sem = "http://marklogic.com/semantics" at "/MarkLogic/semantics.xqy";

	declare function local:triples($uri) {

	  let $bindings := map:map()
	  let $_ := map:put($bindings, "doc", sem:iri( $uri ) )
	  let $results := sem:sparql('CONSTRUCT { $doc ?p ?o } WHERE { $doc ?p ?o }', $bindings)
	  return 
	      sem:rdf-serialize($results, 'rdfxml')
	};

	<People>
	{
	  for $person in /Person
	  return 
	    <Person>  
	      {local:triples($person/URI)}
	      {$person/*}
	    </Person>
	  }
	</People>

The output is:

	<People>
		<Person>
			<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
				<rdf:Description rdf:about="http://person/john">
					<firstname rdf:datatype="http://www.w3.org/2001/XMLSchema#string" xmlns="http://data/">John</firstname>
					<surname rdf:datatype="http://www.w3.org/2001/XMLSchema#string" xmlns="http://data/">Sheridan</surname>
				</rdf:Description>
			</rdf:RDF>
			<URI>http://person/john</URI>
			<Biography>This is John's biography content</Biography>
		</Person>
		<Person>
			<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
				<rdf:Description rdf:about="http://person/marc">
					<firstname rdf:datatype="http://www.w3.org/2001/XMLSchema#string" xmlns="http://data/">Marc</firstname>
					<surname rdf:datatype="http://www.w3.org/2001/XMLSchema#string" xmlns="http://data/">Sturman</surname>
				</rdf:Description>
			</rdf:RDF>
			<URI>http://person/marc</URI>
			<Biography>This is Marc's autobiography content</Biography>
		</Person>
	</People>


#### Loop through triples and return corresponding XML

Next I thought we should do the opposite, this code is basically the same but starts with a SPARQL query and then fetches the corresponding XML, returning the data combined the same way as before.

	xquery version "1.0-ml";
	import module namespace sem = "http://marklogic.com/semantics" at "/MarkLogic/semantics.xqy";
	declare namespace rdf = "http://www.w3.org/1999/02/22-rdf-syntax-ns#";      

	let $bindings := map:map()
	let $results := sem:sparql('CONSTRUCT { ?s ?p ?o } WHERE { ?s ?p ?o }')
	let $rdf := sem:rdf-serialize($results, 'rdfxml')

	return    
	  <People>
	  {
	    for $description in $rdf/rdf:Description
	    return
	      <Person>  
	        <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
	          {$description}
	        </rdf:RDF>
	        {/Person[URI=$description/@rdf:about]/*}
	      </Person>
	  }
	  </People>
	  
We now have the same output:

	<People>
		<Person>
			<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
				<rdf:Description rdf:about="http://person/john">
					<firstname rdf:datatype="http://www.w3.org/2001/XMLSchema#string" xmlns="http://data/">John</firstname>
					<surname rdf:datatype="http://www.w3.org/2001/XMLSchema#string" xmlns="http://data/">Sheridan</surname>
				</rdf:Description>
			</rdf:RDF>
			<URI>http://person/john</URI>
			<Biography>This is John's biography content</Biography>
		</Person>
		<Person>
			<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
				<rdf:Description rdf:about="http://person/marc">
					<firstname rdf:datatype="http://www.w3.org/2001/XMLSchema#string" xmlns="http://data/">Marc</firstname>
					<surname rdf:datatype="http://www.w3.org/2001/XMLSchema#string" xmlns="http://data/">Sturman</surname>
				</rdf:Description>
			</rdf:RDF>
			<URI>http://person/marc</URI>
			<Biography>This is Marc's autobiography content</Biography>
		</Person>
	</People>


#### Can we query for content which matches (some critieria on the XML) AND (some criteria on the RDF)?

Being able to do that would knock this right out of the park, as we could find XML content containing a word (using a full text search) which also has certain metadata properties (using a SPARQL query).

However it is a tricky prosect, since we're dealing with two separate pieces of data in different documents.  For MarkLogic to apply query critiera to separate documents at the same time, it would have to understand the join.  I don't think it can do that.  

But what we can do, is run one query, and then use the results from that query as the input to another query.  This turns out to be simple.  The XQuery below does a SPARQL query for the surname Sturman and retrieves the matching URIs.    Then it does a second search for XML documents that contain the word "autobiography".  We pass extra search criteria into the second search which is the list of all the URIs we got back from the first search.

	xquery version "1.0-ml";
	import module namespace sem = "http://marklogic.com/semantics" at "/MarkLogic/semantics.xqy";
	declare namespace rdf = "http://www.w3.org/1999/02/22-rdf-syntax-ns#";      

	let $results := sem:sparql('CONSTRUCT { ?s <http://data/surname> "Sturman" } WHERE { ?s <http://data/surname> "Sturman" }')
	let $rdf := sem:rdf-serialize($results, 'rdfxml')

	return
	  cts:search( 
	    fn:collection(), 
	    cts:and-query((
	      cts:word-query("autobiography"),
	      cts:or-query((
	        for $uri in $rdf/rdf:Description/@rdf:about
	        return
	          cts:uri-match($uri)
	      ))
	    ))
	  )
	  
This works correctly, finding only documents which match both criteria.  I'm not sure how far it will scale, though, as we could be passing in a huge number of URIs into the criteria for the second search.  

An alternate, but slightly different approach is to run both queries, and filter the results of the second query based on the results from the first query.  The code below loads the URIs returned by the SPARQL query into a map (a look up list) and then filters the results from the second search based on whether the document exists in the map.

	xquery version "1.0-ml";
	import module namespace sem = "http://marklogic.com/semantics" at "/MarkLogic/semantics.xqy";
	declare namespace rdf = "http://www.w3.org/1999/02/22-rdf-syntax-ns#";      

	let $bindings := map:map()
	let $results := sem:sparql('CONSTRUCT { ?s <http://data/surname> "Sturman" } WHERE { ?s <http://data/surname> "Sturman" }')
	let $rdf := sem:rdf-serialize($results, 'rdfxml')

	let $m := map:map()
	let $foo :=
	  for $uri in $rdf/rdf:Description/@rdf:about
	  return 
	    map:put($m, $uri, fn:true())

	return
	  cts:search( fn:collection(), cts:word-query("autobiography"))[ map:get($m, document-uri(.)) ]

The performance of that is probably different, but isn't likely to be good.

#### Hang on there a minute

Like I said before, because the RDF and XML are different documents, it's hard to produce a combined search.  But if they were the same document..

Turns out, MarkLogic have been here before us, and they have a solution.  (this usually happens..)

The SPARQL queries run on a triple index which is produced from the sem:triple elements that we saw stored as XML in the database.  So the thought arises that if we could embed the sem:triple elements inside our XML documents, we might then be able to have a combined query that works on both the XML document and the embedded triples at the same time.

The documentation section here explains how we can embed the sem:triples into XML documents and still have them work in SPARQL queries. https://docs.marklogic.com/guide/semantics/embedded

Furthermore, there's sample code of doing a combined XML+RDF query on the resulting documents, here: https://docs.marklogic.com/guide/semantics/embedded#id_pgfId-917331.  This solution would give the optimum performance for searching both sets at the same time, as it is a single query.  

Unfortunately its slight downsides are that the triples are now stored "inside the XML" and become part of the document content rather than being separated metadata.  It also means that the triples can't be updated using SPARQL update, so if we were to use this, we need to work out our method of publishing triples into the XML content.

Maybe the actual solution, if we need this sort of search, is to go back to the idea of having both an RDF store and periodically publishing the data into the corresponding XML documents.  It's got to be the most performant way to implement a combined search.   I still vote for the original RDF store being inside MarkLogic if possible, though, to reduce the total number of components and the synchronisation issues.

#### Conclusions

* We can put both XML and RDF in a single database
* We can ingest and serialize RDF in a variety of formats
* We can query RDF and combine the results with XML content
* We can query XML and combine the results with RDF content
* We can fetch documents that match both RDF and XML critiera by passing the results of one search into another, with performance limitations
* We can fetch documents that match both RDF and XML critiera by filtering the results of one search using another, with performance limitations
* We can embed RDF inside XML and produce a very performant combined search

Further steps might be:

* Do some more interesting real world examples using Legislation data
* Figure out what solution satisfies the needs of all our different stakeholders 
* Drink tea
