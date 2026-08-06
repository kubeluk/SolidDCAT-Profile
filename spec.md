# Solid DCAT Profile

The Solid Protocol provides mechanisms for managing data in an interoperable and access-controlled manner across decentralized storages, enabling spaces of loosely connected data sources. Solid-based dataspaces follow a bottom-up paradigm, which describes an environment of autonomous, heterogeneous data sources that coexist without prior global integration.

In such an environment, discovery is consiered a fundamental mechanism for establishing relationships between participants' data sources in an incremental fashion. To foster sustainable exchange and reuse of information, i.e., to establish beneficial and meaningful connections between dataspace participants, descriptive *metadata* about the sources and their offered data products need to be captured and managed in a systematic, interoperable and standards-based manner. Ad-hoc or proprietary solutions undermine cross-participant interaction.

## TL;DR

The Solid DCAT Profile offers a discovery mechanism for Solid-based dataspaces by augmenting Solid's RESTful, resource-centric perspective with a dataset-centric, data catalog-style view based on the DCAT standard. By additionally understanding the data catalog itself as a collection of data which allows (1) the data catalog to be managed and interacted with via the Solid Protocol, and (2) the Solid-managed *data catalog* to be decoupled from the Solid-managed *cataloged data*, the Solid DCAT Profile provides the following benefits:
1. Standards-based metadata descriptions via DCAT terms.
2. Follow-your-nose approach to data discovery, as further popularized by the Linked Data Principles.
3. Solid's native authorization mechanism can be re-used for controlling access to individual catalog records allowing for granular privacy preservation.
4. Possibility to independently manage the access control of the catalog itself and the source data described by the catalog.
5. Possibility to individually select which source data to record in the data catalog. 
6. Possibility to distribute a data catalog across multiple Solid storages, allowing for federated catalog deployments.

## Prefixes

```Turtle
@prefix sprof: <http://ex.org/profile/sprof#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
```

## Profile Data Model

The Data Catalog Vocabulary (DCAT) provides standardized RDF terms for describing data catalogs in flexible ways. In some domains or in the context of specific applications, it is necessary or required to be more explicit about a data model's intended usage. The Solid DCAT Profile does exactly that. It adopts many of DCAT's terms and adds constraints of how these terms MUST be used as part of the profile's data model. In the following, we specify these constraints as SHACL shapes.

### Catalog Shape

A `dcat:Catalog` is a curated collection of metadata records describing some `dcat:Resource`. Our profile model defines the following requirements w.r.t. data catalogs:
1. An instance of type `dcat:CatalogRecord` which MAY be linked by a `dcat:Catalog` via `dcat:record` MUST conform with the shape `sprof:CatalogRecordShape`.
2. IRIs linked via `dcat:dataset` MUST conform to exactly one of the two shapes `sprof:DatasetShape` or `sprof:DatasetSeriesShape`.

```Turtle
sprof:CatalogShape
    a sh:NodeShape ;
    sh:targetClass dcat:Catalog ;
    sh:property [
        sh:path dcat:record ;
        sh:node sprof:CatalogRecordShape .
    ] ;
    sh:property [
        sh:path dcat:dataset ;
        sh:xone (
            [
                sh:node sprof:DatasetShape
            ]
            [
                sh:node sprof:DatasetSeriesShape
            ]
        )
    ] .
```

### Catalog Record Shape

A `dcat:CatalogRecord` is a document that contains metadata about `dcat:Resource` instances curated by the data catalog. Our profile model defines the following requirements w.r.t. catalog records:
1. A `dcat:CatalogRecord` document MUST link the main thing that the document is about via `foaf:primaryTopic`.
2. A catalog record's primary topic MUST conform to exactly one of the two shapes `sprof:DatasetShape` or `sprof:DatasetSeriesShape`.

```Turtle
sprof:CatalogRecordShape
    a sh:NodeShape ;
    sh:targetClass dcat:CatalogRecord ;
    sh:property [
        sh:path foaf:primaryTopic ;
        sh:xone (
            [
                sh:node sprof:DatasetShape
            ]
            [
                sh:node sprof:DatasetSeriesShape
            ]
        ) ;
        sh:minCount 1 ;
        sh:maxCount 1
    ] .
```

### Dataset Shape

A `dcat:Dataset` is a collection of data, that might come in one or more representations that make this conceptual notion of a dataset accessible. Our profile model defines the following requirements w.r.t. datasets:
1. A `dcat:Dataset` MUST link an accessible form of representation via `dcat:Distribution` that conforms with the shape `sprof:DistributionShape`.
2. A `dcat:Dataset` MUST link a blank node or an IRI via `dcat:theme` to state the dataset's main category.

```Turtle
sprof:DatasetShape
    a sh:NodeShape ;
    sh:targetClass dcat:Dataset ;
    sh:property [
        sh:path dcat:distribution ;
        sh:node sprof:DistributionShape ;
		sh:minCount 1
    ] ;
	sh:property [
        sh:path dcat:theme ;
        sh:nodeKind sh:BlankNodeOrIRI ;
        sh:minCount 1
    ] .
```

### Dataset Series Shape

A `dcat:DatasetSeries` is a dataset that represents a collection of datasets that are published separately, but share some characteristics that group them. Our profile model defines the following requirements w.r.t. dataset series:
1. As each instance of `dcat:DatasetSeries` is an instance of `dcat:Dataset`, a dataset series MUST conform with the shape `sprof:DatasetShape`.
2. IRIs linked via `dcat:seriesMember` MUST conform to exactly one of the two shapes `sprof:DatasetShape` or `sprof:DatasetSeriesShape`.

```Turtle
sprof:DatasetSeriesShape
    a sh:NodeShape ;
    sh:targetClass dcat:DatasetSeries ;
    sh:node sprof:DatasetShape ;
    sh:property [
        sh:path dcat:seriesMember ;
        sh:xone (
            [
                sh:node sprof:DatasetShape
            ]
            [
                sh:node sprof:DatasetSeriesShape
            ]
        ) ;
    ] .
```

### Distribution Shape

A `dcat:Distribution` is an accessible form of a dataset such as a downloadable file. Our profile model defines the following requirements w.r.t. distributions:
1. A distribution MUST link at most one downloadable file via `dcat:downloadURL`.
2. A distribution's media type MAY be linked via `dcat:mediaType` and MUST either be stated as a string or as an IRI.
3. A distribution MAY indicate the model, schema, ontology, view or profile that this representation of a dataset conforms to via `dcterms:conformsTo` wich MUST be a SHACL shape or any other type of description given as blank node or IRI.

```Turtle
sprof:DistributionShape
    a sh:NodeShape ;
    sh:targetClass dcat:Distribution ;
    sh:property [
        sh:path dcat:downloadURL ;
        sh:nodeKind sh:IRI ;
        sh:maxCount 1
    ] ;
    sh:property [
        sh:path dcat:mediaType ;
        sh:xone (
            [
                sh:nodeKind sh:Literal ;
				sh:datatype xsd:string
            ]
            [
                sh:nodeKind sh:IRI
            ]
        ) ;
    ] ;
	sh:or (
		[
			sh:property [
				sh:path dcterms:conformsTo ;
				sh:node shsh:NodeShapeShape
			]
		]
		[
			sh:property [
				sh:path dcterms:conformsTo ;
				sh:nodeKind sh:BlankNodeOrIRI ;
			]
		]
	) .
```

## Profile Catalog Management Model

The profile's data model, introduced in the previous section, stated the shapes that define how Solid DCAT data catalogs are represented.
The profile's catalog management model, introduced in this section, defines requirements and operations for Creating, Reading, Updating, and Deleting (CRUD) data catalogs and their managed resources.
Data catalogs that follow this specification MAY curate any type of information resource

### Hosting a Solid DCAT Catalog

We define a Solid DCAT data catalog to be a collection of `dcat:CatalogRecord` documents that are referenced via `dcat:record` by a `dcat:Catalog` instance within a data catalog document. In the following, we view these two types of documents as managed by a #StorageServer: 
- A data catalog document MUST be a Linked Data Platform RDF Source (LDP-RS) that is the representation of a RDF graph that conforms to the shape `sprof:CatalogShape`.
- A catalog record document MUST be a Linked Data Platform RDF Source (LDP-RS) that is the representation of a RDF graph that conforms to the shape `sprof:CatalogRecordShape`.

### Adding a Resource to the Catalog

To record a new resource in the catalog:
1. A new document of type `dcat:CatalogRecord` MUST be created on a #StorageServer.
2. A new triple MUST be added to the #DataCatalog document that links the #DataCatalog and the #CatalogRecord via `dcat:record`.
3. A new #Dataset (instance of type `dcat:Dataset`) MUST be created as a secondary resource by referencing the primary #CatalogRecord resource and an additional fragment identifier component.
4. IF the #SolidResource is of type `ldp:Container`, THEN the secondary #Dataset resource MUST further be of type `dcat:DatasetSeries`.
5. The #CatalogRecord MUST link the secondary #Dataset resource via `foaf:primaryTopic`.
6. A new #Distribution (instance of type `dcat:Distribution`) MUST be created as a secondary resource by referencing the primary #CatalogRecord and an additional fragment identifier component.
7. The #Dataset resource MUST link the #Distribution resource via `dcat:distribution`.
8. The #Distribution resource MUST link the #SolidResource via `dcat:downloadURL`.
9. IF the #SolidResource is part of a containment triple (`ldp:contains`) in _object position_ AND the resource in subject position is in the set of the recorded resources in the #DataCatalog, THEN the #Dataset resource describing the #SolidResource MUST link the #Dataset resource describing the resource which is _above_ the #SolidResource in the containment hierarchy via `dcat:inSeries`.
10. IF the #SolidResource is part of a containment triple (`ldp:contains`) in _subject position_ AND the resource in object position is in the set of the recorded resources in the #DataCatalog, THEN the #Dataset resource describing the #SolidResource MUST link the #Dataset resource describing the resource which is _below_ the #SolidResource in the containment hierarchy via `dcat:hasMember`.

### Removing a resource from the catalog

> HTTP DELETE on the catalog record + PATCH DELETE of record triple from catalog document

### Updating a catalog record

> link Solid + recommend HTTP PATCH on catalog resource

### Discovery mechanisms

A #DataCatalog serves as a common entry point for conducting discovery procedures. The #DataCatalog described in this document follows the Linked Data Principles and the RESTful architecture-style of the Web. It thus MAY be linked by another source. A source referencing a #DataCatalog MUST use the predicate `http://purl.org/sdp/terms#catalog`.

> **_TODO_**: add sequence diagram from paper

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