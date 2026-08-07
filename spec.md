# Solid DCAT Profile

The Solid Protocol provides mechanisms for managing data in an interoperable and access-controlled manner across decentralized storage servers , enabling spaces of loosely connected data sources. Solid-based dataspaces follow a bottom-up paradigm, which describes an environment of autonomous, heterogeneous data sources that coexist without prior global integration.

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

> **_TODO_**: Create PURL URI

```Turtle
@prefix sprof: <http://ex.org/profile/sprof#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
```

## Profile Data Model

The Data Catalog Vocabulary (DCAT) provides standardized RDF terms for describing data catalogs in flexible ways. In some domains or in the context of specific applications, it is necessary or required to be more explicit about a data model's intended usage. The Solid DCAT Profile does exactly that. It adopts DCAT's data model in parts and states how Solid DCAT data catalogs are represented that augment Solid storages and curate their Solid-managed resources. In the following, we specify the added constraints on the data model as SHACL shapes.

### Catalog Shape

A `dcat:Catalog` is a curated collection of metadata records describing some `dcat:Resource`. Our profile data model defines the following requirements w.r.t. data catalogs:
1. Resources linked via `dcat:record` MUST conform to the shape `sprof:CatalogRecordShape`.
2. Resources linked via `dcat:dataset` MUST conform to exactly one of the two shapes `sprof:DatasetShape` or `sprof:DatasetSeriesShape`.

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

A `dcat:CatalogRecord` is a metadata document about `dcat:Resource` instances curated by a Solid DCAT data catalog. Our profile model defines the following requirements w.r.t. catalog records:
1. A `dcat:CatalogRecord` document MUST link the resource that the document's metadata is about via `foaf:primaryTopic`.
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

A `dcat:Dataset` is a collection of data, which comes in one or more representations that make the conceptual notion of a dataset accessible. Our profile model defines the following requirements w.r.t. datasets:
1. A `dcat:Dataset` MUST link an accessible form of representation via `dcat:Distribution` that conforms to the shape `sprof:DistributionShape`.
2. A `dcat:Dataset` MUST link a theme via `dcat:theme` to state the dataset's main category.
3. Resources linked via `dcat:inSeries` MUST conform to the shape `sprof:DatasetSeriesShape`.

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
        sh:minCount 1
    ] ;
	sh:property [
		sh:path dcat:inSeries ;
		sh:node sprof:DatasetSeriesShape
	] .
```

### Dataset Series Shape

A `dcat:DatasetSeries` is a dataset which represents a collection of datasets that are published separately, but share some characteristics that group them. Our profile model defines the following requirements w.r.t. dataset series:
1. As each instance of `dcat:DatasetSeries` is an instance of `dcat:Dataset`, a dataset series MUST conform to the shape `sprof:DatasetShape`.
2. Resources linked  via `dcat:seriesMember` MUST conform to exactly one of the two shapes `sprof:DatasetShape` or `sprof:DatasetSeriesShape`.

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
1. A distribution MUST link one downloadable file via `dcat:downloadURL`.
2. A distribution MUST link one media type via `dcat:mediaType`.
3. A distribution MUST indicate at least one model, schema, ontology, view or profile that this representation of a dataset conforms to via `dcterms:conformsTo`.

```Turtle
sprof:DistributionShape
    a sh:NodeShape ;
    sh:targetClass dcat:Distribution ;
    sh:property [
        sh:path dcat:downloadURL ;
        sh:nodeKind sh:IRI ;
		sh:minCount 1 ;
        sh:maxCount 1
    ] ;
    sh:property [
        sh:path dcat:mediaType ;
		sh:minCount 1 ;
        sh:maxCount 1
    ] ;
	sh:property [
		sh:path dcterms:conformsTo ;
		sh:minCount 1
	] .
```

## Profile Catalog Management Model

The profile's data model, introduced in the previous section, stated the shapes that define how Solid DCAT data catalogs are represented.
The profile's catalog management model, introduced in this section, defines the catalog's bahaviour and how cataloged resources are managed.

### Hosting a Solid DCAT Catalog

A Solid DCAT data catalog is a collection of two types of documents: (1) One root catalog document, and (2) several catalog record documents.

A root catalog document MUST conform to the shape `sprof:CatalogShape`.

A catalog record document MUST conform to the shape `sprof:CatalogRecordShape`.

These two types of documents are Solid-managed information resources:
- The behaviour of a data catalog root document MUST corresponds to a Linked Data Platform RDF Source (LDP-RS).
- The behaviour of a catalog record document MUST correspond to a Linked Data Platform RDF Source (LDP-RS).

The root data catalog document and all its referenced catalog record documents MAY be managed by the same Solid storage server. Nevertheless, federated deployment scanarios are also possible, where the documents are distributed across various Solid storage servers.

### Adding a Resource to a Catalog

There is a 1-1 correspondence between Solid DCAT catalog record documents and information resources managed by a Solid storage server.

A Solid-managed resource that was added to a Solid DCAT data catalog MUST be augmented by exactly one catalog record that conforms to the shape `sprof:CatalogRecordShape`.

Clients can add a new Solid-managed resource to a Solid DCAT data catalog by performing the following operations:

1. Creating a corresponding catalog record:

A catalog record MUST be created on a Solid storage server.

When the resource is a `ldp:Container` an instance of type `dcat:DatasetSeries` MUST be created as a secondary resource by referencing the primary catalog record resource and an additional fragment identifier component. The catalog record MUST reference the dataset series via `foaf:primaryTopic`.

When the resource is *not* a `ldp:Container` an instance of type `dcat:Dataset` MUST be created as a secondary resource by referencing the primary catalog record resource and an additional fragment identifier component. The catalog record MUST reference the dataset via `foaf:primaryTopic`.

The fragment identifier component for a dataset / dataset series MAY be `#ds`.

An instance of type `dcat:Distribution` MUST be created as a secondary resource by referencing the primary catalog record resource and an additional fragment identifier component.

The fragement identifier component for a distribution MAY be `#dist`.

The created dataset / dataset series MUST link the created distribution via `dcat:distribution`.

The created distribution MUST link the Solid-managed resource via `dcat:downloadURL`.

> What about `dcat:inSeries` and `dcat:hasMember`?
> IF the #SolidResource is part of a containment triple (`ldp:contains`) in _object position_ AND the resource in subject position is in the set of the recorded resources in the #DataCatalog, THEN the #Dataset resource describing the #SolidResource MUST link the #Dataset resource describing the resource which is _above_ the #SolidResource in the containment hierarchy via `dcat:inSeries`.
> IF the #SolidResource is part of a containment triple (`ldp:contains`) in _subject position_ AND the resource in object position is in the set of the recorded resources in the #DataCatalog, THEN the #Dataset resource describing the #SolidResource MUST link the #Dataset resource describing the resource which is _below_ the #SolidResource in the containment hierarchy via `dcat:hasMember`.

2. Linking the new catalog record

The new catalog record MUST be linked by the root catalog document via `dcat:record`.

A client MAY perform step 1 by a HTTP `POST` request to a URI path ending with `/` as defined in the Solid Protocol (§5.3).

A client MAY perform step 2 by a HTTP `PATCH` request to the root catalog document as defined in the Solid Protocol (§5.3.1).

### Reading a Catalog Resource

Clients retrieve a representation of a resource's state using HTTP `GET`, `HEAD` and `OPTIONS`, as defined in the Solid Protocol (§5.2).

### Updating a Catalog Resource

Solid DCAT catalog resources MAY be modified using N3 patches as defined in the Solid Protocol (§5.3.1).

### Removing a Catalog Resource

> (optional) PATCH DELETE of dcat:hasMember triple when dcat:inSeries triple is present in corresponding catalog record + HTTP DELETE on the catalog record + PATCH DELETE of record triple from catalog document

### Discovery mechanisms

A Solid DCAT data catalog serves as a common entry point for conducting discovery procedures. The Solid DCAT Profile follows the Linked Data Principles and the RESTful architecture-style of the Web.

A Solid DCAT data catalog MAY be referenced by another resource.

A resource referencing a Solid DCAT data catalog MUST use the predicate `http://purl.org/sdp/terms#catalog`.

> **_TODO_**: add sequence diagram from paper

#### Discovering the catalog

Without limiting generality, two modes of operation could be considered:
1. A #DataCatalog MAY be discovered via the #StorageServer that hosts data recorded by the #DataCatalog by following the `http://purl.org/sdp/terms#catalog` links found in the #StorageServer's description resource.
2. A #DataCatalog MAY be discovered via an agent's WebID document by following the `http://purl.org/sdp/terms#catalog` links that point to a #StorageServer hosting a #DataCatalog which curates datasets offered by the agent.

#### Discovering catalog resources

> "just follow the damn links..."

### Controling access to a catalog record

> link Solid / WAC

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