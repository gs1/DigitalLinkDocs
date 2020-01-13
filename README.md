- [Quick Start Documentation for various GS1 Digital Link tools and libraries](#quick-start-documentation-for-various-gs1-digital-link-tools-and-libraries)
  - [Overall aim](#overall-aim)
  - [The basics](#the-basics)
  - [Constructing the GS1 Digital Link URI](#constructing-the-gs1-digital-link-uri)
  - [Accessing multiple links](#accessing-multiple-links)
    - [HTTP-only](#http-only)
    - [linkType=all](#linktypeall)
  - [Requesting a specific link](#requesting-a-specific-link)
  - [Downloading all available links in bulk](#downloading-all-available-links-in-bulk)

# Quick Start Documentation for various GS1 Digital Link tools and libraries

GS1 conformant resolvers act primarily as redirection services and, with the exception of one command, do not usually return content directly. They are defined in chapter 8 of version 1.1 of the GS1 Digital Link standard (see [current draft](https://www.gs1.org/sites/default/files/docs/gsmp/gs1_digital_link_1.1_comrev_version_786.pdf)).
## Overall aim
The overall aim of a GS1 conformant resolver is that applications can resolve a GS1 identifier, such as a GTIN, SSCC or GLN, and be redirected to one or more resources relevant to the identified item.
## The basics
At its most basic, the structure of a GS1 Digital Link URI is:

`https://id.gs1.org/{primaryKeyName}/{primaryKeyValue}`

For example:

The GTIN for the (fictitious) Dal Giardino Risotto Rice with Mushrooms is `09506000134352` so at its simplest, using the ‘API’ on the id.gs1.org resolver is simply

`GET https://id.gs1.org/gtin/9506000134352`
## Constructing the GS1 Digital Link URI
GS1 Digital Link HTTP URIs are an expression of the full GS1 system of global identifiers for trade items, shipments, locations and more. This can become complex but you can construct the HTTP URI using the [GS1 Digital link toolkit](https://github.com/gs1/GS1DigitalLinkToolkit.js) – a JavaScript library that has its own documentation and an [online demo page](https://gs1.github.io/GS1DigitalLinkToolkit.js/) . The output of any GS1 barcode can be fed to that toolkit, along with the base URI of your resolver, and the corresponding GS1 Digital Link URI is returned. 
## Accessing multiple links
Redirection to the resource you most likely want is just the default behaviour. The resolver may have multiple links associated with a given identifier and these can be discovered in two ways:
### HTTP-only
You can do an HTTP HEAD request on any GS1 Digital Link URI and suppress your client’s redirection. The resolver will include the full list of available links in its Link response header.
The key elements of the HTTP trace for the Dal Giardino example are as follows:

`HTTP/1.1 307 Temporary Redirect`

`Location: https://dalgiardino.com/risotto-rice-with-mushrooms/`

`Link: <https://dalgiardino.com/risotto-rice-with-mushrooms/>; rel="pip"; type="text/html"; hreflang="en"; title="Product information page", <https://dalgiardino.com/mushroom-squash-risotto/>; rel="recipeinfo"; type="text/html"; hreflang="en"; title="Recipe website", <https://dalgiardino.com/where-to-buy/>; rel="hasretailers"; type="text/html"; hreflang="en"; title="Retailers", <https://dalgiardino.com/about/>; rel="productsustainabilityinfo"; type="text/html"; hreflang="en"; title="sustainability and recycling"`

(2019-11-15 Note [Issue 4](https://github.com/gs1/GS1_DigitalLink_Resolver_CE/issues/4))

The 307 response and the Location header tell you the destination of the redirect that would be applied with a regular GET. The Link header includes a list of options (separated by commas), each of which has a number of fields (separated by semicolons):
1.	The target URL
2.	The link relationship type
3.	The media type of the target resource
4.	The language of the target resource
5.	A human-readable title for the link

For the example response above, the information contained in the Link: response is as follows:

| Target URL | Link relationship type | Media type | Language | Title |
| ---------- | ---------------------- | ---------- | -------- | ----- |
| https://dalgiardino.com/risotto-rice-with-mushrooms | pip | text/html | en | Product information page |
| https://dalgiardino.com/mushroom-squash-risotto/ | recipeInfo | text/html | en | Recipe website |
| https://dalgiardino.com/where-to-buy | hasRetailers | text/html | en | Has retailers |
| https://dalgiardino.com/about/ | productSustainabilityInfo | text/html | en | Product sustainability information |

### linkType=all
The same list of available links can be obtained as either a JSON object or an HTML page if you use the value ‘all’ for the linkType parameter. For example:

`https://id.gs1.org/gtin/9506000134352?linkType=all`

The JSON object returned to clients specifying that media type in the HTTP request is identical to the one included in and processed by the HTML page.
The nesting is a result of the flexibility of the system. Resolvers allow the differentiation of links associated with a particular GS1 key according to:
1. The link relationship type (known in GS1 Digital Link as simply the ‘link type’)
2. The language of the target resource
3. 	The media type of the target resource
4.	A further value called context.
The value space (e.g. permitted / defined values) for the context parameter is not defined in the GS1 Digital Link standard but is defined separately for each resolver in the Resolver Description File that can be found as a JSON object at `/.well-known/gs1resolver` for any GS1 conformant resolver (see the [id.gs1.org resolver example](https://id.gs1.org/.well-known/gs1resolver)). The values can be expressed either as an enumerated list (in an array provided as the value of `supportedContextValuesEnumerated` and/or by naming one or more external lists of values as the value of the `supportedContextValuesExternal` property).
## Requesting a specific link
Rather than redirecting to the default link, a resolver will redirect requests for a specific link type if one is available. For example:

`https://id.gs1.org/gtin/9506000134352?linkType=recipeInfo`

will redirect to a recipe for that product rather than the product information page, which is the default. 
GS1 maintains a list of link types as part of its [Web vocabulary](https://mh1.eu/voc/?show=linktypes) but you can get a quick listing of the link types in use in a particular resolver by consulting its [Resolver Description File](https://id.gs1.org/.well-known/gs1resolver). You can get a list of all the GS1 link types as a JSON object from https://mh1.eu/voc/linktypes.php (link to be updated soon)

## Downloading all available links in bulk
It's possible to grab all available links as a bulk download. See the [separate page](https://gs1.github.io/DigitalLinkDocs/bulkdownload.html) for more information. Please note that this experimental feature is not part of GS1 Digital Link version 1.1 (draft) and is not yet filly implemented (it needs to include pagination, for example) but it will be considered for inclusion in version 1.2. It is implemented in the [community edition resolver](https://github.com/gs1/GS1_DigitalLink_Resolver_CE). 

# Where's the community?
Please report any bugs with any of the software by raising an issue in the relevant repository. If you want to ask questions and generally be part of the broader developer community around GS1 Digital Link, please join our [Google group](https://groups.google.com/forum/#!forum/gs1-digital-link-developers)

# Running a pilot
The resolver at `id.gs1.org` is available for pilots and tests. GS1 Member Organisations and GCP licensees may request an account on the resolver. Contact <a href="mailto:phil.archer@gs1.org">Phil Archer</a> for details. 
