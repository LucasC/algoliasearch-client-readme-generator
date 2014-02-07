Algolia Search API Client for C#
==================



[Algolia Search](http://www.algolia.com) is a search API that provides hosted full-text, numerical and faceted search.
Algolia’s Search API makes it easy to deliver a great search experience in your apps & websites providing:

 * REST and JSON-based API
 * search among infinite attributes from a single searchbox
 * instant-search after each keystroke
 * relevance & popularity combination
 * typo-tolerance in any language
 * faceting
 * 99.99% SLA
 * first-class data security

This C# client let you easily use the Algolia Search API from your backend. It wraps [Algolia's REST API](http://www.algolia.com/doc/rest_api).(Compatible with .NET 4.5, SL4+, WP7.5+, Windows Store)



Table of Content
-------------
**Get started**

1. [Setup](#setup)
1. [Quick Start](#quick-start)


**Commands reference**

1. [Search](#search)

1. [Add a new object](#add-a-new-object-in-the-index)
1. [Update an object](#update-an-existing-object-in-the-index)
1. [Get an object](#get-an-object)
1. [Delete an object](#delete-an-object)
1. [Index settings](#index-settings)
1. [List indexes](#list-indexes)
1. [Delete an index](#delete-an-index)
1. [Clear an index](#clear-an-index)
1. [Wait indexing](#wait-indexing)
1. [Batch writes](#batch-writes)
1. [Security / User API Keys](#security--user-api-keys)
1. [Copy or rename an index](#copy-or-rename-an-index)
1. [Backup / Retrieve all index content](#backup--retrieve-all-index-content)
1. [Logs](#logs)



Setup
-------------
To setup your project, follow these steps:

 1. In you project, open the "Package Manager Console" (Tools → Library Package Manager → Package Manager Console)
 2. Enter `Install-Package Algolia.Search` in the Package Manager Console
 3. Initialize the client with your ApplicationID and API-Key. You can find all of them on [your Algolia account](http://www.algolia.com/users/edit).

```csharp
using Algolia.Search;

AlgoliaClient client = new AlgoliaClient("YourApplicationID", "YourAPIKey");
```

Quick Start
-------------

This quick start is a 30 seconds tutorial where you can discover how to index and search objects.

Without any prior-configuration, you can index [500 contacts](https://github.com/algolia/algoliasearch-client-csharp/blob/master/contacts.json) in the ```contacts``` index with the following code:
```csharp
// Load JSON file
StreamReader re = File.OpenText("contacts.json");
JsonTextReader reader = new JsonTextReader(re);
JArray batch = JArray.Load(reader);
// Add objects 
Index index = client.InitIndex("contacts");
await index.AddObjects(batch);
```

You can then start to search for a contact firstname, lastname, company, ... (even with typos):
```csharp
// search by firstname
System.Diagnostics.Debug.WriteLine(await index.Search(new Query("jimmie")));
// search a firstname with typo
System.Diagnostics.Debug.WriteLine(await index.Search(new Query("jimie")));
// search for a company
System.Diagnostics.Debug.WriteLine(await index.Search(new Query("california paint")));
// search for a firstname & company
System.Diagnostics.Debug.WriteLine(await index.Search(new Query("jimmie paint")));
```

Settings can be customized to tune the search behavior. For example you can add a custom sort by number of followers to the already good out-of-the-box relevance:
```csharp
await index.SetSettings(JObject.Parse(@"{""customRanking"":[""desc(followers)""]}"));
```

You can also configure the list of attributes you want to index by order of importance (first = most important):
```csharp
await index.SetSettings(JObject.Parse(@"{""attributesToIndex"":[""lastname"", ""firstname"",
                                                                ""company"", ""email"", ""city""]}"));
```

Since the engine is designed to suggest results as you type, you'll generally search by prefix. In this case the order of attributes is very important to decide which hit is the best:
```csharp
System.Diagnostics.Debug.WriteLine(await index.Search(new Query("or")));
System.Diagnostics.Debug.WriteLine(await index.Search(new Query("jim")));
```




Search
-------------
 **Opening note:** If you are building a web application, you may be more interested in using our [javascript client](https://github.com/algolia/algoliasearch-client-js) to send queries. It brings two benefits: (i) your users get a better response time by avoiding to go through your servers, and (ii) it will offload your servers of unnecessary tasks.

To perform a search, you just need to initialize the index and perform a call to the search function.

You can use the following optional arguments on Query class:

### Query parameters

#### Full Text Search parameters

 * **SetQueryString**: (string) The instant-search query string, all words of the query are interpreted as prefixes (for example "John Mc" will match "John Mccamey" and "Johnathan Mccamey"). If no query parameter is set, retrieves all objects.
 * **SetQueryType**: select how the query words are interpreted, it can be one of the following value:
  * **PREFIX_ALL**: all query words are interpreted as prefixes,
  * **PREFIX_LAST**: only the last word is interpreted as a prefix (default behavior),
  * **PREFIX_NONE**: no query word is interpreted as a prefix. This option is not recommended.
 * **SetOptionalWords**: a string that contains the list of words that should be considered as optional when found in the query. The list of words is comma separated.
 * **SetMinWordSizeToAllowOneTypo**: the minimum number of characters in a query word to accept one typo in this word.<br/>Defaults to 3.
 * **SetMinWordSizeToAllowTwoTypos**: the minimum number of characters in a query word to accept two typos in this word.<br/>Defaults to 7.

#### Pagination parameters

 * **SetPage**: (integer) Pagination parameter used to select the page to retrieve.<br/>Page is zero-based and defaults to 0. Thus, to retrieve the 10th page you need to set `page=9`
 * **SetNbHitsPerPage**: (integer) Pagination parameter used to select the number of hits per page. Defaults to 20.

#### Geo-search parameters

 * **AroundLatitudeLongitude(float, float, int)**: search for entries around a given latitude/longitude.<br/>You specify the maximum distance in meters with the **radius** parameter (in meters).<br/>At indexing, you should specify geoloc of an object with the _geoloc attribute (in the form `{"_geoloc":{"lat":48.853409, "lng":2.348800}}`)
 * **AroundLatitudeLongitude(flot, float, int, int)**: search for entries around a given latitude/longitude with a given precision for ranking (for example if you set precision=100, two objects that are distant of less than 100m will be considered as identical for "geo" ranking parameter).

 * **InsideBoundingBox**: search entries inside a given area defined by the two extreme points of a rectangle (defined by 4 floats: p1Lat,p1Lng,p2Lat,p2Lng).<br/>For example ``.<br/>At indexing, you should specify geoloc of an object with the _geoloc attribute (in the form `{"_geoloc":{"lat":48.853409, "lng":2.348800}}`)

#### Parameters to control results content

 * **SetAttributesToRetrieve**: The list of object attributes you want to retrieve (let you minimize the answer size). By default, all attributes are retrieved. You can also use `*` to retrieve all values when an **attributesToRetrieve** setting is specified for your index.
 * **SetAttributesToHighlight**: The list of attributes you want to highlight according to the query. If an attribute has no match for the query, the raw value is returned. By default all indexed text attributes are highlighted. You can use `*` if you want to highlight all textual attributes. Numerical attributes are not highlighted. A matchLevel is returned for each highlighted attribute and can contain:
  * **full**: if all the query terms were found in the attribute,
  * **partial**: if only some of the query terms were found,
  * **none**: if none of the query terms were found.
 * **SetAttributesToSnippet**: The list of attributes to snippet alongside the number of words to return (syntax is `attributeName:nbWords`). By default no snippet is computed.
 * **GetRankingInfo**: if set to true, the result hits will contain ranking information in **_rankingInfo** attribute.


#### Numeric search parameters
 * **SetNumericFilters**: a string that contains the list of numeric filters you want to apply separated by a comma. The syntax of one filter is `attributeName` followed by `operand` followed by `value`. Supported operands are `<`, `<=`, `=`, `>` and `>=`. 
 You can have multiple conditions on one attribute like for example `numericFilters=price>100,price<1000`. You can also use a string array encoding (for example `numericFilters: ["price>100","price<1000"]`).

#### Category search parameters
 * **SetTagFilters**: filter the query by a set of tags. You can AND tags by separating them by commas. To OR tags, you must add parentheses. For example, `tags=tag1,(tag2,tag3)` means *tag1 AND (tag2 OR tag3)*. You can also use a string array encoding, for example `tagFilters: ["tag1",["tag2","tag3"]]` means *tag1 AND (tag2 OR tag3)*.<br/>At indexing, tags should be added in the **_tags** attribute of objects (for example `{"_tags":["tag1","tag2"]}`). 

#### Faceting parameters
 * **SetFaceFilters**: filter the query by a list of facets. Facets are separated by commas and each facet is encoded as `attributeName:value`. For example: `facetFilters=category:Book,author:John%20Doe`. You can also use a string array encoding (for example `["category:Book","author:John%20Doe"]`).
 * **SetFacets**: List of object attributes that you want to use for faceting. <br/>Attributes are separated with a comma (for example `"category,author"` ). You can also use a JSON string array encoding (for example `["category","author"]` ). Only attributes that have been added in **attributesForFaceting** index setting can be used in this parameter. You can also use `*` to perform faceting on all attributes specified in **attributesForFaceting**.

#### Distinct parameter
 * **EnableDistinct**: If set to YES, enable the distinct feature (disabled by default) if the `attributeForDistinct` index setting is set. This feature is similar to the SQL "distinct" keyword: when enabled in a query with the `distinct=1` parameter, all hits containing a duplicate value for the attributeForDistinct attribute are removed from results. For example, if the chosen attribute is `show_name` and several hits have the same value for `show_name`, then only the best one is kept and others are removed.

```csharp
Index index = client.InitIndex("contacts");
res = await index.Search(new Query("query string"));
System.Diagnostics.Debug.WriteLine(res);
res = await index.Search(new Query("query string").
    SetAttributesToRetrieve(new string[] {"firstname","lastname"}).
    SetNbHitsPerPage(50));
System.Diagnostics.Debug.WriteLine(res);
```

The server response will look like:

```javascript
{
  "hits": [
    {
      "firstname": "Jimmie",
      "lastname": "Barninger",
      "objectID": "433",
      "_highlightResult": {
        "firstname": {
          "value": "<em>Jimmie</em>",
          "matchLevel": "partial"
        },
        "lastname": {
          "value": "Barninger",
          "matchLevel": "none"
        },
        "company": {
          "value": "California <em>Paint</em> & Wlpaper Str",
          "matchLevel": "partial"
        }
      }
    }
  ],
  "page": 0,
  "nbHits": 1,
  "nbPages": 1,
  "hitsPerPage": 20,
  "processingTimeMS": 1,
  "query": "jimmie paint",
  "params": "query=jimmie+paint&attributesToRetrieve=firstname,lastname&hitsPerPage=50"
}
```


Add a new object in the Index
-------------

Each entry in an index has a unique identifier called `objectID`. You have two ways to add en entry in the index:

 1. Using automatic `objectID` assignement, you will be able to retrieve it in the answer.
 2. Passing your own `objectID`

You don't need to explicitely create an index, it will be automatically created the first time you add an object.
Objects are schema less, you don't need any configuration to start indexing. The settings section provide details about advanced settings.

Example with automatic `objectID` assignement:

```csharp
var res = await index.AddObject(JObject.Parse(@"{""firstname"":""Jimmie"", 
                                                 ""lastname"":""Barninger""}"));
System.Diagnostics.Debug.WriteLine("objectID=" + res["objectID"]);           
```

Example with manual `objectID` assignement:

```charp
var res = await index.AddObject(JObject.Parse(@"{""firstname"":""Jimmie"", 
                                                 ""lastname"":""Barninger""}"), "myID");
System.Diagnostics.Debug.WriteLine("objectID=" + res["objectID"]);
```

Update an existing object in the Index
-------------

You have two options to update an existing object:

 1. Replace all its attributes.
 2. Replace only some attributes.

Example to replace all the content of an existing object:

```csharp
await index.SaveObject(JObject.Parse(@"{""firstname"":""Jimmie"", 
                                        ""lastname"":""Barninger"", 
                                        ""city"":""New York"",
                                        ""objectID"":""myID""}"));
```

Example to update only the city attribute of an existing object:

```csharp
await index.PartialUpdateObject(JObject.Parse(@"{""city"":""San Francisco"", 
                                                 ""objectID"":""myID""}"));
```

Get an object
-------------

You can easily retrieve an object using its `objectID` and optionnaly a list of attributes you want to retrieve (using comma as separator):

```csharp
// Retrieves all attributes
var res = await index.GetObject("myID");
System.Diagnostics.Debug.WriteLine(res);
// Retrieves firstname and lastname attributes
res = await index.GetObject("myID", new String[] {"firstname", "lastname"});
System.Diagnostics.Debug.WriteLine(res);
// Retrieves only the firstname attribute
res = await index.GetObject("myID", new String[] { "firstname" });
System.Diagnostics.Debug.WriteLine(res);
```

Delete an object
-------------

You can delete an object using its `objectID`:

```csharp
await index.DeleteObject("myID");
```

Index Settings
-------------

You can retrieve all settings using the `GetSettings` function. The result will contains the following attributes:


#### Indexing parameters
 * **attributesToIndex**: (array of strings) the list of fields you want to index.<br/>If set to null, all textual of your objects are indexed, but you should update it to get optimal results.<br/>This parameter has two important uses:
  * *Limit the attributes to index*.<br/>For example if you store a binary image in base64, you want to store it and be able to retrieve it but you don't want to search in the base64 string.
  * *Control part of the ranking*.<br/>(see the ranking parameter for full explanation) Matches in attributes at the beginning of the list will be considered more important than matches in attributes further down the list. In one attribute, matching text at the beginning of the attribute will be considered more important than text after, you can disable this behavior if you add your attribute inside `unordered(AttributeName)`, for example `attributesToIndex: ["title", "unordered(text)"]`.
 * **attributesForFaceting**: (array of strings) The list of fields you want to use for faceting. All strings in the attribute selected for faceting are extracted and added as a facet. If set to null, no attribute is used for faceting.
 * **attributeForDistinct**: The attribute name used for the `Distinct` feature. This feature is similar to the SQL "distinct" keyword: when enabled in query with the `distinct=1` parameter, all hits containing a duplicate value for this attribute are removed from results. For example, if the chosen attribute is `show_name` and several hits have the same value for `show_name, then only the best one is kept and others are removed.
 * **ranking**: (array of strings) controls the way results are sorted.<br/>We have six available criteria: 
  * **typo**: sort according to number of typos,
  * **geo**: sort according to decreassing distance when performing a geo-location based search,
  * **proximity**: sort according to the proximity of query words in hits,
  * **attribute**: sort according to the order of attributes defined by attributesToIndex,
  * **exact**: 
    * if the user query contains one word: sort objects having an attribute that is exactly the query word before others. For example if you search for the "V" TV show, you want to find it with the "V" query and avoid to have all popular TV show starting by the v letter before it.
    * if the user query contains multiple words: sort according to the number of words that matched exactly (and not as a prefix).
  * **custom**: sort according to a user defined formula set in **customRanking** attribute.<br/>The standard order is ["typo", "geo", "proximity", "attribute", "exact", "custom"]
 * **customRanking**: (array of strings) lets you specify part of the ranking.<br/>The syntax of this condition is an array of strings containing attributes prefixed by asc (ascending order) or desc (descending order) operator.
For example `"customRanking" => ["desc(population)", "asc(name)"]`  
 * **queryType**: Select how the query words are interpreted, it can be one of the following value:
  * **prefixAll**: all query words are interpreted as prefixes,
  * **prefixLast**: only the last word is interpreted as a prefix (default behavior),
  * **prefixNone**: no query word is interpreted as a prefix. This option is not recommended.
 * **slaves**: The list of indexes on which you want to replicate all write operations. In order to get response times in milliseconds, we pre-compute part of the ranking during indexing. If you want to use different ranking configurations depending of the use-case, you need to create one index per ranking configuration. This option enables you to perform write operations only on this index, and to automatically update slave indexes with the same operations.

#### Default query parameters (can be overwrite by query)
 * **minWordSizefor1Typo**: (integer) the minimum number of characters to accept one typo (default = 3).
 * **minWordSizefor2Typos**: (integer) the minimum number of characters to accept two typos (default = 7).
 * **hitsPerPage**: (integer) the number of hits per page (default = 10).
 * **attributesToRetrieve**: (array of strings) default list of attributes to retrieve in objects. If set to null, all attributes are retrieved.
 * **attributesToHighlight**: (array of strings) default list of attributes to highlight. If set to null, all indexed attributes are highlighted.
 * **attributesToSnippet**: (array of strings) default list of attributes to snippet alongside the number of words to return (syntax is 'attributeName:nbWords')<br/>By default no snippet is computed. If set to null, no snippet is computed.
 * **highlightPreTag**: (string) Specify the string that is inserted before the highlighted parts in the query result (default to "&lt;em&gt;").
 * **highlightPostTag**: (string) Specify the string that is inserted after the highlighted parts in the query result (default to "&lt;/em&gt;").
 * **optionalWords**: (array of strings) Specify a list of words that should be considered as optional when found in the query.

You can easily retrieve settings or update them:

```csharp
var res = await index.GetSettings();
System.Diagnostics.Debug.WriteLine(res);
```

```csharp
await index.SetSettings(JObject.Parse(@"{""customRanking"":[""desc(followers)""]}"));
```

List indexes
-------------
You can list all your indexes with their associated information (number of entries, disk size, etc.) with the `listIndexes` method:

```csharp
var result = await client.ListIndexes();
System.Diagnostics.Debug.WriteLine(res);
```

Delete an index
-------------
You can delete an index using its name:

```csharp
await client.DeleteIndex("contacts");
```

Clear an index
-------------
You can delete the index content without removing settings and index specific API keys with the clearIndex command:

```csharp
await index.ClearIndex();
```

Wait indexing
-------------

All write operations return a `taskID` when the job is securely stored on our infrastructure but not when the job is published in your index. Even if it's extremely fast, you can easily ensure indexing is complete using the `waitTask` method on the `taskID` returned by a write operation. 

For example, to wait for indexing of a new object:
```csharp
var res = await index.AddObject(JObject.Parse(@"{""firstname"":""Jimmie"", 
                                                 ""lastname"":""Barninger""}"), "myID");
await index.WaitTask(res["taskID"].ToString());
```


If you want to ensure multiple objects have been indexed, you can only check the biggest taskID.

Batch writes
-------------

You may want to perform multiple operations with one API call to reduce latency.
We expose three methods to perform batch:
 * `AddObjects`: add an array of object using automatic `objectID` assignement
 * `SaveObjects`: add or update an array of object that contains an `objectID` attribute
 * `PartialUpdateObjects`: partially update an array of objects that contain an `objectID` attribute (only specified attributes will be updated, other will remain unchanged)

Example using automatic `objectID` assignement:
```csharp
List<JObject> objs = new List<JObject>();
objs.Add(JObject.Parse(@"{""firstname"":""Jimmie"", 
                          ""lastname"":""Barninger""}"));
objs.Add(JObject.Parse(@"{""firstname"":""Warren"", 
                          ""lastname"":""Speach""}"));
var res = await index.AddObjects(objs);
System.Diagnostics.Debug.WriteLine(res);
```

Example with user defined `objectID` (add or update):
```csharp
List<JObject> objs = new List<JObject>();
objs.Add(JObject.Parse(@"{""firstname"":""Jimmie"", 
                          ""lastname"":""Barninger"",
                          ""objectID"":""myID1""}"));
objs.Add(JObject.Parse(@"{""firstname"":""Warren"", 
                          ""lastname"":""Speach"",
                          ""objectID"": ""myID2""}"));
var res = await index.SaveObjects(objs);
System.Diagnostics.Debug.WriteLine(res);
```

Example that update only the `firstname` attribute:
```csharp
List<JObject> objs = new List<JObject>();
objs.Add(JObject.Parse(@"{""firstname"":""Jimmie"", 
                          ""objectID"":""myID1""}"));
objs.Add(JObject.Parse(@"{""firstname"":""Warren"", 
                          ""objectID"": ""myID2""}"));
var res = await index.PartialUpdateObjects(objs);
System.Diagnostics.Debug.WriteLine(res);
```



Security / User API Keys
-------------

The admin API key provides full control of all your indexes. 
You can also generate user API keys to control security. 
These API keys can be restricted to a set of operations or/and restricted to a given index.

To list existing keys, you can use `listUserKeys` method:
```csharp
// Lists global API Keys
var keys = await client.ListUserKeys();
// Lists API Keys that can access only to this index
keys = await index.ListUserKeys();
```

Each key is defined by a set of rights that specify the authorized actions. The different rights are:
 * **search**: allows to search,
 * **browse**: allow to retrieve all index content via the browse API,
 * **addObject**: allows to add/update an object in the index,
 * **deleteObject**: allows to delete an existing object,
 * **deleteIndex**: allows to delete index content,
 * **settings**: allows to get index settings,
 * **editSettings**: allows to change index settings.

Example of API Key creation:
```csharp
// Creates a new global API key that can only perform search actions
var res = await client.AddUserKey(new String[] { "search" });
System.Diagnostics.Debug.WriteLine("Key: " + res["key"]);
// Creates a new API key that can only perform search action on this index
res = await index.AddUserKey(new String[] { "search" });
System.Diagnostics.Debug.WriteLine("Key: " + res["key"]);
```

You can also create an API Key with advanced restrictions:

 * Add a validity period: the key will be valid only for a specific period of time (in seconds),
 * Specify the maximum number of API calls allowed from an IP address per hour. Each time an API call is performed with this key, a check is performed. If the IP at the origin of the call did more than this number of calls in the last hour, a 403 code is returned. Defaults to 0 (no rate limit). This parameter can be used to protect you from attempts at retrieving your entire content by massively querying the index.

 * Specify the maximum number of hits this API key can retrieve in one call. Defaults to 0 (unlimited). This parameter can be used to protect you from attempts at retrieving your entire content by massively querying the index.

```csharp
// Creates a new global API key that is valid for 300 seconds
var res = await client.AddUserKey(new String[] { "search" }, 300, 0, 0);
System.Diagnostics.Debug.WriteLine("Key: " + res["key"]);
// Creates a new index specific API key valid for 300 seconds, with a rate limit of 100 calls per hour per IP and a maximum of 20 hits
res = await index.AddUserKey(new String[] { "search" }, 300, 100, 20);
System.Diagnostics.Debug.WriteLine("Key: " + res["key"]);
```

Get the rights of a given key:
```csharp
// Gets the rights of a global key
var res = await client.GetUserKeyACL("f420238212c54dcfad07ea0aa6d5c45f");
// Gets the rights of an index specific key
res = await index.GetUserKeyACL("71671c38001bf3ac857bc82052485107");
```

Delete an existing key:
```csharp
// Deletes a global key
await client.DeleteUserKey("f420238212c54dcfad07ea0aa6d5c45f");
// Deletes an index specific key
await index.DeleteUserKey("71671c38001bf3ac857bc82052485107");
```

Copy or rename an index
-------------

You can easily copy or rename an existing index using the `copy` and `move` commands.
**Note**: Move and copy commands overwrite destination index.

```csharp
// Rename MyIndex in MyIndexNewName
await client.MoveIndex("MyIndex", "MyIndexNewName");
// Copy MyIndex in MyIndexCopy
await client.CopyIndex("MyIndex", "MyIndexCopy");
```

The move command is particularly useful is you want to update a big index atomically from one version to another. For example, if you recreate your index `MyIndex` each night from a database by batch, you just have to:
 1. Import your database in a new index using [batches](#batch-writes). Let's call this new index `MyNewIndex`.
 1. Rename `MyNewIndex` in `MyIndex` using the move command. This will automatically override the old index and new queries will be served on the new one.

```csharp
// Rename MyNewIndex in MyIndex (and overwrite it)
await client.MoveIndex("MyNewIndex", "MyIndex");
```

Backup / Retrieve all index content
-------------

You can retrieve all index content for backup purpose of for analytics using the browse method. 
This method retrieve 1000 objects by API call and support pagination.

```csharp
// Get first page
await index.Browse(0);
// Get second page
await index.Browse(1);
```

Logs
-------------

You can retrieve the last logs via this API. Each log entry contains: 
 * Timestamp in ISO-8601 format
 * Client IP
 * Request Headers (API-Key is obfuscated)
 * Request URL
 * Request method
 * Request body
 * Answer HTTP code
 * Answer body
 * SHA1 ID of entry

You can retrieve the logs of your last 1000 API calls and browse them using the offset/length parameters:
 * ***offset***: Specify the first entry to retrieve (0-based, 0 is the most recent log entry). Default to 0.
 * ***length***: Specify the maximum number of entries to retrieve starting at offset. Defaults to 10. Maximum allowed value: 1000.

```csharp
// Get last 10 log entries
await client.GetLogs();
// Get last 100 log entries
await client.GetLogs(0, 100);
```





