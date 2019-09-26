# Database Design

Blaze uses [Datomic](https://www.datomic.com) as its main database because Datomic offers in-memory graph traversal and includes a tansaction history out of the box. By comparision to a relational database, Datomic works completely different. The data model of Datomic are facts like RDF triples.

| Entity | Attribute | Value |
| :--- | :--- | :--- |
| sally | likes | pizza |

The above table shows one fact, namely that sally likes pizza. A Datomic database just consists of a big set of such facts. The entity part of such a fact is an identifier which is common over all facts of an entity like sally. That said, it is important to note that entities don't have an explicit representation in Datomic. Entities only exist implecitely through there facts. The attribute part describes in which role the value relates to the entity. Attributes are schema elements in Datomic and have to exist in the database before a fact can be added. The schema entity of an attribute, among other things, describes the data type of the value.

## Basic FHIR Resource Representation

In order to represent FHIR resources in Datomic, the [structure definition](http://www.hl7.org/fhir/structuredefinition.html) of each resource is used to generate Datomic schema entities \(attributes\). Each structure definition consists of a list of [element definitions](http://www.hl7.org/fhir/elementdefinition.html). Each element definition is translated into mostly one Datomic attribute.

Two properties of element definitions are relevant for the schema generation, the cardinality and weather it is a choice-typed element or not. The following sections describe the schema elements of the three different combinations of that two properties.

### Single-Valued Single-Typed Elements

The most basic data elements are single-valued and single-typed. One example is the data element `Patient.active`. A Patient resource with only that data elements looks like this:

```json
{
  "resourceType": "Patient",
  "active": true
}
```

The element definition `Patient.active` is shown below as JSON, stripped from irrelevant properties.

```json
{
  "path": "Patient.active",
  "max": "1",
  "type": [
    {
      "code": "boolean"
    }
  ]
}
```

The first property is `path` which uniquely identifies an element definition and describes its position as element directly under the `Patient` resource. Next, the `max` property describes the cardinality of an element. Here `1` means cardinality one and `*` means cardinality many. The `type` property describes the type which is boolean in this case. The `type` property is multi-valued itself to support choice-typed elements.

The corresponding [Datomic attribute definition](https://docs.datomic.com/on-prem/schema.html) is shown below as EDN.

```clojure
{:db/ident :Patient/active
 :db/valueType :db.type/boolean
 :db/cardinality :db.cardinality/one}
```

{% hint style="info" %}
[EDN](https://github.com/edn-format/edn) \(Extensible Data Notation\) has many things in common with [JSON](http://json.org) \(JavaScript Object Notation\). The main feature EDN introduces, are extensible data types. Among them, EDN has one build-in data type, the keyword which is used in the Datomic attribute definition. Keywords start with a colon and consist of two parts, a namespace and a name. For example in `:db/ident`,`db` is the namespace and `ident` the name.
{% endhint %}

In the above schema definition, the attribute defined is called `:Patient/active,` the data type \(`:db/valueType`\) of the attribute is a boolean and the cardinality is one. 

### Multi-Valued Single-Typed Elements

Elements can be multi-valued so there value is represented as a JSON array. One example of a multi-valued element can be found in the `ServiceRequest` resource:

```json
{
  "resourceType": "ServiceRequest",
  "instantiatesUri": [
      "http://foo.de",
      "http://bar.de"
  ]
}
```

The `instantiatesUri` element is defined as shown below:

```json
{
  "path": "ServiceRequest.instantiatesUri",
  "max": "*",
  "type": [
    {
      "code": "uri"
    }
  ]
}
```

The important new value is the `*` in `max`, which means cardinality many. The Datomic attribute definition looks like this:

```clojure
{:db/ident :ServiceRequest/instantiatesUri
 :db/valueType :db.type/string
 :db/cardinality :db.cardinality/many}
```

Cardinality many in Datomic simply means that more than one fact can exist for a given entity and attribute. The facts, resulting from the above example JSON are the following:

| Entity | Attribute | Value |
| :--- | :--- | :--- |
| 1 | `:ServiceRequest/instantiatesUri` | `"http://foo.de"` |
| 1 | `:ServiceRequest/instantiatesUri` | `"http://bar.de"` |

{% hint style="warning" %}
There is one issue representing multi-valued data elements by just using Datomic attributes with cardinality many. Such Datomic attributes have set semantics, so that the order isn't preserved. However for some data elements, the order has a meaning. See [Issue \#15](https://github.com/life-research/blaze/issues/15) for more information.
{% endhint %}

### Choice-Typed Elements

An example of a choice-typed element is `Patient.deceased[x]`. Choice-typed elements are always single-valued. The chosen data type can vary between resources. An example:

```json
{
  "resourceType": "Patient",
  "deceasedBoolean": true
}
```

The element definition:

```json
{
  "path": "Patient.deceased[x]",
  "max": "1",
  "type": [
    {
      "code": "boolean"
    },
    {
      "code": "dateTime"
    }
  ]
}
```

For choice typed elements, multiple Datomic attributes are created: one for each data type and one which holds the actual data type used in a given resource. For `Patient.deceased[x]` the following three attributes are created:

```clojure
{:db/ident :Patient/deceased
 :db/valueType :db.type/ref
 :db/cardinality :db.cardinality/one}

{:db/ident :Patient/deceasedBoolean
 :db/valueType :db.type/boolean
 :db/cardinality :db.cardinality/one}

{:db/ident :Patient/deceasedDateTime
 :db/valueType :db.type/bytes
 :db/cardinality :db.cardinality/one}
```

The attribute `:Patient/deceased` is actually a reference to the attribute of the chosen type which is `:Patient/deceasedBoolean` in this case. The following pattern can be used to access the actual value:

```clojure
(let [resource (d/entity db resource-eid)]
  (when-let [attr (:Patient/deceased resource)]
    (attr resource)))
```

Where `(:Patient/deceased resource)` will either return `:Patient/deceasedBoolean`, `:Patient/deceasedDateTime` or `nil`.

### Non-Primitive Single-Valued Single-Typed Elements

The next class of element definitions are those with non-primitive data types. One example is the `Patient.maritalStatus`.

```json
{
  "path": "Patient.maritalStatus",
  "max": "1",
  "type": [
    {
      "code": "CodeableConcept"
    }
  ]
}
```

In its elemend definition the data type is `CodeableConcept` which is a non-primitive data type. Non-primitive data types consit of multiple elements themself and are represented as a JSON object rather one of the primitive JSON data types. In Datomic such non-primitive types are represented as entities \(like the patient resource itself\) and the element having that data type becomes a reference type which can refer to entities holding the data of the non-primitive type.

The Datomic attribute definition of the above `Patient.martialStatus` element is the following:

```text
{:db/ident :Patient/maritalStatus
 :db/valueType :db.type/ref
 :db/cardinality :db.cardinality/one}
```

The only difference is the data type of the attribute defined which is `:db.type/ref` the reference type.

The [CodeableConcept](https://www.hl7.org/fhir/datatypes.html#CodeableConcept) data type itself has elements like `CodeableConcept.text` which is a primitive string element and its Datomic attribute definition is the following:

```text
{:db/ident :CodeableConcept/text
 :db/valueType :db.type/string
 :db/cardinality :db.cardinality/one}
```

#### Example

With all that schema definitions in hand, the following patient resource written in JSON:

```json
{
  "resourceType": "Patient",
  "id": "0",
  "active": true,
  "maritalStatus": {
    "text": "Married"
  }
}
```

will consist of the following two entities in Datomic:

```text
{:db/id 1
 :Patient/id "0"
 :Patient/active true
 :Patient/martialStatus 2}
 
{:db/id 2
 :CodeableConcept/text "Married"}
```

The first entity represents the `Patient` resource while the second entity represents the non-primitive data type `CodeableConcept`. This representation of entities with the help of EDN maps is already a shorter representation of the following facts:

| Entity | Attribute | Value |
| :--- | :--- | :--- |
| 1 | `:Patient/id` | "0" |
| 1 | `:Patient/active` | true |
| 1 | `:Patient/martialStatus` | 2 |
| 2 | `:CodeableConcept/text` | "Married" |

In the above table the four facts, which represent the `Patient` resource from the above JSON are shown. Of particilar interest is the fact which the attribute `:Patient/martialStatus` which refers to the entity with the identifier `2`, which itself holds the fact with attribute `:CodeableConcept/text` and value "Married".

### Choice-Typed Elements

Data elements can be choice typed, which means that every resource can choose one out of the type choices for that element. One common example is the element `Observation.value[x]`, which has various type choices like `Quantity` and `string`. In JSON the type is appended title-cased to the name of the element.

```json
{
  "resourceType": "Observation"
  "valueQuantity": {
    "value": 1
    "unit": "m"
  }
}
```

In the above example, the JSON key is `valueQuantity` referring to the `Quantity` type for that resource. As choice-typed elements can be only single-valued, only one type can be choosen for each resource.

The translation of a choice-typed data element into a Datomic schema creates one attribute definition for each type choice and one extra attribute holding the type choosen for the particular resource. For `Observation.value[x]` the following attribute definition are created \(only showing two types\):

```text
{:db/ident :Observation/valueQuantity
 :db/valueType :db.type/ref
 :db/cardinality :db.cardinality/one}
 
{:db/ident :Observation/valueString
 :db/valueType :db.type/string
 :db/cardinality :db.cardinality/one}
 
{:db/ident :Observation/value
 :db/valueType :db.type/ref
 :db/cardinality :db.cardinality/one}
```

The above `Observation` resource is converted to the following Datomic entities:

```text
{:db/id 1
 :Observation/value :Observation/valueQuantity
 :Observation/valueQuantity 2}
 
{:db/id 2
 :Quantity/value 1
 :Quantity/unit "m"}
```

The additional `:Observation/value` attribute is nescessary, because otherwise, graph traversal from `Observation` to its value had to search for the concrete value attribute choosen.

### Special Id-Typed Elements

[Logical id elements](https://www.hl7.org/fhir/resource-definitions.html#Resource.id) are handled a bit special. They act as primary key for resources and so must be unique and indexed. The element definition of a patient id is the following \(only relevant properties\):

```json
{
  "path": "Patient.id",
  "max": "1",
  "type": [
    {
      "code": "id"
    }
  ]
}
```

The only thing that makes the element definition different from the `Patient.active` definition is the data type. Here it's `id` instead of `boolean`. Element definitions with a single `id` data type are converted into Datomic attribute definition with one additional key-value pair. The above `Patient.id` translates to:

```text
{:db/ident :Patient/id
 :db/valueType :db.type/string
 :db/cardinality :db.cardinality/one
 :db/unique :db.unique/identity}
```

The additional key-value pair consists of the key `:db/unique` and the value `:db.unique/identity`. This information tells Datomic to keep sure, that all values of the attribute `:Patient/id` are unique in the sense that only one entity can hold a fact with a given value.

### Special Version Attribute

Every Datomic entity, representing a resource has an additional `:version` attribute. The version attribute is used for having a version number at all, creation mode, marking resources as deleted and optimistic locking. It has the following definition:

```text
{:db/ident :version
 :db/valueType :db.type/long
 :db/cardinality :db.cardinality/one}
```

As a first simplified approximation, the version attribute will be incremented with every update to a resource. The inital version of a `Patient` resource could look like this:

#### Version Number

```text
{:db/id 1
 :Patient/id "4c50eb52-d362-43fc-81fc-738f4193b510"
 :version 0
 :Patient/active false}
```

were the first update, which activates the patient would result in:

```text
{:db/id 1
 :Patient/id "4c50eb52-d362-43fc-81fc-738f4193b510"
 :version 1
 :Patient/active true}
```

Because Datomic keeps a history of all changed facts, the version number will be a fact which changes on every update of a resource. That ever changing version number allows to refer to the transaction which was responsible for the last update on a resource.

{% hint style="info" %}
In Datomic transactions are reified as they are entities themself. Every transaction leads to a entity with at least one attribute the `:db/txInstant`, which makes the transaction time always discoverable. Every fact which will be added carries a reference to its transaction. To be more precise, facts aren't triples \(entity, attribute, value\) the are quadruples \(entity, attribute, value, transaction\) in Datomic.
{% endhint %}

The Clojure function returning the transaction of the last update of a resource is the following:

```text
(defn last-transaction
  [{eid :db/id :as resource}]
  (let [db (d/entity-db resource)]
    (d/entity db (:tx (first (d/datoms db :eavt eid :version))))))
```

In that function, the `d/datoms` function is used to obtain the fact of the given resource entity and the `:version` attribute. The fact contains the transaction identifier named `:tx` which is used to obtain the transaction entity.

{% hint style="info" %}
The `d/datoms` function allows direct access to indexed facts. Datomic has several indices were the `:eavt` \(Entity, Attribute, Value, Transaction\) index is the main one. That index contains all facts of a database ordered by their entity id, attribute id, value and transaction id.
{% endhint %}

In the [read interaction](https://www.hl7.org/fhir/http.html#read), the last transaction is used to fill the `meta` data elements `versionId` and `lastUpdated`. Especially the last updated time has not to be stored in the resource entity itself because that information is available through the transaction.

In the [history interaction](https://www.hl7.org/fhir/http.html#history), the `:version` attribute is used to obtain an ordered history of all updates from the history database of Datomic.

{% hint style="info" %}
In Datomic the term database doesn't refer to something like a traditional database, which is expected to change continuously. Like in [persistent data structures](https://en.wikipedia.org/wiki/Persistent_data_structure), a Datomic database is immutable, more like a complex value that never changes. Each transaction will take the current database value and produce a new one. The history database is a special database which contains all facts, which were added and retracted over time. Thoose facts have a fiveth component which states whether the fact was added or retracted.
{% endhint %}

The following function returns all transactions of a resource in the order they appeared with the newest first:

```text
(defn transaction-history
  [db eid]
  (eduction
    (filter :added)
    (map #(d/entity db (:tx %)))
    (d/datoms (d/history db) :eavt eid :version)))
```

Like in the `last-transaction` function, the `d/datoms` function is used to obtain the facts with the `:version` attribute of a particilar resource. But here, the history database is used and so all additions and retractions on the `:version` attribute are returned, ordered by the value of the version itself.

{% hint style="warning" %}
Because the history interaction is supposed to return the newest version first and the `:eavt` index is ordered ascendingly, it is necessary to decrement the version number instead of incrementing it.
{% endhint %}

#### Creation Mode

Resources can be created through the [create](https://www.hl7.org/fhir/http.html#create) or the [update interaction](https://www.hl7.org/fhir/http.html#update). If the create interaction is used, the logical identifier of a resource is choosen by the server, were the update interaction allows the client to specify the logical identifier. Furthermore the create interaction uses the HTTP POST method and a type-level URL while the update interaction uses PUT and an instance level URL.

In the history bundle returned by the history interaction, each entry has to specify the HTTP request which lead to the particular version. At first it seems natural to store the information, whether create or update was used to create the first version of a resource,  in the transaction entity itself. But the transaction interaction allows creating multiple resources during one transaction so it would be necessary to iterate all ...

