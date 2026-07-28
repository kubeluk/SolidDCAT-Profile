# Solid DCAT Profile

normative Mapping zwischen Solid/LDP-Ressourcen und DCAT-Ressourcen

This section defines the Solid DCAT Profile model. The profile describes resources and container resources using DCAT concepts while keeping the DCAT metadata resources separate from the resources they describe. DCAT resources act as metadata proxies and do not change the type or semantics of the underlying Solid resources.

## Profile Model

When a Solid resource is to be described by a dcat:Dataset then MUST exists a second Solid resource which is a dcat:CatalogRecord and that dataset MUST link to a dcat:Distribution, which itself links to the first Solid resource using dcat:downloadURL.

(andersherum? data model vom catalog, data model vom record als datenmodellierungs kapitel und dann kapitel hosting catalog resource, beides beschrieben, wenn ein resource soll auf storage server gehostet werden damit access control verfügbar ist)

Wenn ein record über solid resource zum katalog hinzugefügt werden soll dann müssen links existieren zwischen Dokumenten  
When a Solid resource is recorded in a catalog then there MUST exists the three links between the documents

Catalog Document und record document als Classes of Product, catalog besteht aus zwei arten von documents
## Hosting Model

A #DataCatalog is essentially a set of #CatalogRecord documents of type `dcat:CatalogRecord` that are referenced by a #DataCatalog of type `dcat:Catalog`. The Solid DCAT profile dictates such documents to be managed by a #StorageServer:
- A #DataCatalog document MUST be managed by a #StorageServer.
- A #CatalogRecord document MUST be managed by a #StorageServer.

This implies:
- A #DataCatalog document MUST be a Linked Data Platform RDF Source (LDP-RS).
- A #CatalogRecord document MUST be a Linked Data Platform RDF Source (LDP-RS).

The following operations MUST be performed to record a #SolidResource in a #DataCatalog:
1. A new #CatalogRecord document MUST be created on a #StorageServer.
2. A new triple MUST be added to the #DataCatalog document that links the #DataCatalog and the #CatalogRecord via `dcat:record`.
3. A new #Dataset (instance of type `dcat:Dataset`) MUST be created as a secondary resource by referencing the primary #CatalogRecord resource and an additional fragment identifier component.
4. IF the #SolidResource is of type `ldp:Container` the secondary #Dataset resource MUST further be of type `dcat:DatasetSeries`.
5. The #CatalogRecord MUST link the secondary #Dataset resource via `foaf:primaryTopic`.
6. A new #Distribution (instance of type `dcat:Distribution`) MUST be created as a secondary resource by referencing the primary #CatalogRecord and an additional fragment identifier component.
7. The #Dataset resource MUST link the #Distribution resource via `dcat:distribution`.
8. 
### Example

Assume the storage `<http://ex.org/s/>` on a #StorageServer that stores one container which contains one document:

```Turtle
@prefix ldp: <http://www.w3.org/ns/dcat#> .
@prefix pim: <http://www.w3.org/ns/pim/space#> .
@base <http://ex.org/s/> .

<>
	a pim:Storage ;
	ldp:contains <c/> .
```

```Turtle
@prefix ldp: <http://www.w3.org/ns/dcat#> .
@base <http://ex.org/s/c/> .

<>
	a ldp:Container ;
	ldp:contains <1> .
```

The following example shows the state of all resources on the #StorageServer when the #DataCatalog itself is hosted on the same #StorageServer and each #SolidResource was recorded in the #DataCatalog. The #DataCatalog document's location on the #StorageServer is up to the implementor's choice. For this example, the location `<http://ex.org/s/cat/alog>` was chosen:

```Turtle
@prefix ldp: <http://www.w3.org/ns/dcat#> .
@prefix pim: <http://www.w3.org/ns/pim/space#> .
@base <http://ex.org/s/> .

<>
	a pim:Storage ;
	ldp:contains <c/> .
```

```Turtle
@prefix ldp: <http://www.w3.org/ns/dcat#> .
@base <http://ex.org/s/c/> .

<>
	a ldp:Container ;
	ldp:contains <1> .
```

```Turtle
@prefix ldp: <http://www.w3.org/ns/dcat#> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@base <http://ex.org/s/c/1> .

<>
	a ldp:Resource .

<#lukas>
	a foaf:Person ;
	foaf:firstName "Lukas" .
```

```Turtle
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@base <http://ex.org/cat/alog> .

<#it>
	a dcat:Catalog ;
	dcat:record <r1>, <r2> .
```

```Turtle
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@base <http://ex.org/cat/r1> .

<> 
	a dcat:CatalogRecord ;
	foaf:primaryTopic <#ds> .
	
<#ds>
	a dcat:DatasetSeries ;
	dcat:distribution <#dist1> ;
	dcat:hasMember <r2#ds> .
	
<#dist1>
	a dcat:Distribution ;
	dcat:downloadURL <../s/c/> .
```

```Turtle
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@base <http://ex.org/cat/r2> .

<>
	a dcat:CatalogRecord ;
	foaf:primaryTopic <#ds> .
	
<#ds>
	a dcat:Dataset ;
	dcat:distribution <#dist1> ;
	dcat:inSeries <r1#ds> .
	
<#dist1>
	a dcat:Distribution ;
	dcat:downloadURL <../s/c/1> .
```
## Discovery Model
A #DataCatalog serves as a common entry point for conducting discovery procedures. The #DataCatalog described in this document follows the Linked Data Principles and the RESTful architecture-style of the Web. It thus MAY be linked by another source. A source referencing a #DataCatalog SHOULD use the predicate `http://purl.org/sdp/terms#catalog`.

Without limiting generality, two modes of operation could be considered:
1. A #DataCatalog MAY be discovered via the #StorageServer that hosts data recorded by the #DataCatalog by following the `http://purl.org/sdp/terms#catalog` links found in the #StorageServer's description resource.
2. A #DataCatalog MAY be discovered via an agent's WebID document by following the `http://purl.org/sdp/terms#catalog` links that point to a #StorageServer hosting a #DataCatalog which curates datasets offered by the agent.

## Notes
- do we need to have a separate name for the #StorageServer that hosts the #DataCatalog documents, i.e. #CatalogDocument and #CatalogRecord?
	- The rationale is that a #DataCatalog is not necessarily hosted on the #StorageServer which contains #SolidResource s that are recorded in the #DataCatalog. Likewise, a #DataCatalog might record #SolidResource s that are hosted on #StorageServer s which are different from the #StorageServer on which the #DataCatalog is hosted.