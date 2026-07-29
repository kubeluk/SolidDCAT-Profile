# Solid DCAT Profile

normative Mapping zwischen Solid/LDP-Ressourcen und DCAT-Ressourcen

This section defines the Solid DCAT Profile model. The profile describes resources and container resources using DCAT concepts while keeping the DCAT metadata resources separate from the resources they describe. DCAT resources act as metadata proxies and do not change the type or semantics of the underlying Solid resources.

## Profile Model

> **_NOTE:_** Simply describe and show the shapes graph. Talk about which properties MUST be given. Talk about domains and ranges of properties. E.g.: "the `dcat:downloadURL`MUST link to a #SolidResource"

> Catalog Document und Record document als Classes of Product

## "Instantiating the Catalog Graph" / Managing the Catalog / Interaction Model

A #DataCatalog is essentially a set of #CatalogRecord documents of type `dcat:CatalogRecord` that are referenced by a #DataCatalog of type `dcat:Catalog` via `dcat:record`. The Solid DCAT profile dictates #CatalogRecord documents to be managed by a #StorageServer:
- A #DataCatalog document MUST be managed by a #StorageServer.
- A #CatalogRecord document MUST be managed by a #StorageServer.

This implies:
- A #DataCatalog document MUST be a Linked Data Platform RDF Source (LDP-RS).
- A #CatalogRecord document MUST be a Linked Data Platform RDF Source (LDP-RS).

> **_NOTE:_** It might only be relevant for catalog records to be managed by a storage server? I guess we do not need to make an assumption about the existence of a data catalog record?

### Adding a resource to the catalog

The following operations MUST be performed to record a #SolidResource in a #DataCatalog:
1. A new #CatalogRecord document MUST be created on a #StorageServer.
2. A new triple MUST be added to the #DataCatalog document that links the #DataCatalog and the #CatalogRecord via `dcat:record`.
3. A new #Dataset (instance of type `dcat:Dataset`) MUST be created as a secondary resource by referencing the primary #CatalogRecord resource and an additional fragment identifier component.
4. IF the #SolidResource is of type `ldp:Container`, THEN the secondary #Dataset resource MUST further be of type `dcat:DatasetSeries`.
5. The #CatalogRecord MUST link the secondary #Dataset resource via `foaf:primaryTopic`.
6. A new #Distribution (instance of type `dcat:Distribution`) MUST be created as a secondary resource by referencing the primary #CatalogRecord and an additional fragment identifier component.
7. The #Dataset resource MUST link the #Distribution resource via `dcat:distribution`.
8. The #Distribution resource MUST link the #SolidResource via `dcat:downloadURL`.
9. IF the #SolidResource is part of a containment triple (`ldp:contains`) in _object position_ AND the resource in subject position is in the set of the recorded resources in the #DataCatalog, THEN the #Dataset resource describing the #SolidResource MUST link the #Dataset resource describing the resource which is _above_ the #SolidResource in the containment hierarchy via `dcat:inSeries`.
10. IF the #SolidResource is part of a containment triple (`ldp:contains`) in _subject position_ AND the resource in object position is in the set of the recorded resources in the #DataCatalog, THEN the #Dataset resource describing the #SolidResource MUST link the #Dataset resource describing the resource which is _below_ the #SolidResource in the containment hierarchy via `dcat:hasMember`.

> **_NOTE:_** Do we need to have a separate name for the #StorageServer that hosts the #DataCatalog documents, i.e. #CatalogDocument and #CatalogRecord? The idea is that a #DataCatalog is not necessarily hosted on the #StorageServer which contains #SolidResource resources that are recorded in the #DataCatalog. Likewise, a #DataCatalog might record #SolidResource resources that are hosted on #StorageServer instances which are different from the #StorageServer on which the #DataCatalog is hosted.

### Removing a resource from the catalog

> HTTP DELETE on the catalog record + PATCH DELETE of record triple from catalog document

### Updating a catalog record

> link Solid + recommend HTTP PATCH on catalog resource

### Discovery mechanisms

A #DataCatalog serves as a common entry point for conducting discovery procedures. The #DataCatalog described in this document follows the Linked Data Principles and the RESTful architecture-style of the Web. It thus MAY be linked by another source. A source referencing a #DataCatalog MUST use the predicate `http://purl.org/sdp/terms#catalog`.

> add sequence diagram from paper

#### Discovering the catalog

Without limiting generality, two modes of operation could be considered:
1. A #DataCatalog MAY be discovered via the #StorageServer that hosts data recorded by the #DataCatalog by following the `http://purl.org/sdp/terms#catalog` links found in the #StorageServer's description resource.
2. A #DataCatalog MAY be discovered via an agent's WebID document by following the `http://purl.org/sdp/terms#catalog` links that point to a #StorageServer hosting a #DataCatalog which curates datasets offered by the agent.

#### Discovering catalog resources

> "just follow the damn links..."

### Controling access to a catalog record

> link Solid / WAC + describe benefits aus paper

## Example

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