- [Quick Start Documentation for various GS1 Digital Link tools and libraries](#quick-start-documentation-for-various-gs1-digital-link-tools-and-libraries)
  - [Overall aim](#overall-aim)
  - [The basics](#the-basics)
  - [Constructing the GS1 Digital Link URI](#constructing-the-gs1-digital-link-uri)
  - [Accessing multiple links](#accessing-multiple-links)
    - [HTTP-only](#http-only)
    - [linkType=all](#linktypeall)
  - [Requesting a specific link](#requesting-a-specific-link)
  - [Downloading all available links in bulk](#downloading-all-available-links-in-bulk)
  - [Demo app](#demo-app)
- [Underlying principles](#underlying-principles)
- [Where's the community?](#wheres-the-community)
- [Running a pilot](#running-a-pilot)

# Quick Start Documentation for various GS1 Digital Link tools and libraries

GS1 conformant resolvers act primarily as redirection services and, with the exception of one command, do not usually return content directly. They are defined in chapter 7 of version 1.1 of the [GS1 Digital Link standard](https://www.gs1.org/docs/Digital-Link/GS1_Digital_link_Standard_i1.1.pdf).

Please note that all GS1 standards carry a [legal disclaimer](https://ref.gs1.org/gs1/standards-disclaimer/).
## Overall aim
The overall aim of a GS1 conformant resolver is that applications can resolve a GS1 identifier, such as a GTIN, SSCC or GLN, and be redirected to one or more resources relevant to the identified item. There's a [9 minute video](https://youtu.be/H2idDJeH3o4) that provides an introduction to the different layers that comprise GS1 Digital Link.
## The basics
At its most basic, the structure of a GS1 Digital Link URI is:

`https://{resolver}/{primaryKeyName}/{primaryKeyValue}`

For example:

The GTIN for the (fictitious) Dal Giardino Risotto Rice with Mushrooms is `09506000134352` so at its simplest, using the ‘API’ on the id.gs1.org resolver is simply

`GET https://id.gs1.org/01/9506000134352`

The `01` is the name of the GS1 identifier all of which are numeric. Within GS1, these are known as 'application identifiers', a full list of which is [maintained by GS1](https://www.gs1.org/standards/barcodes/application-identifiers). `01` is the most common as it is the application identifier for the GTIN.
## Constructing the GS1 Digital Link URI
GS1 Digital Link HTTP URIs are an expression of the full GS1 system of global identifiers for trade items, shipments, locations and more. You can get a good idea of how this works in the [2D barcode generator](https://gs1.github.io/2d-barcode-generator/).

<!-- (removed for duration of GS1 US hackathon) This can become complex but you can construct the HTTP URI using the [GS1 Digital link toolkit](https://github.com/gs1/GS1DigitalLinkCompressionPrototype) – a JavaScript library that has its own documentation and an [online demo page](https://gs1.github.io/GS1DigitalLinkCompressionPrototype/). The output of any GS1 barcode can be fed to that toolkit, along with the base URI of your resolver, and the corresponding GS1 Digital Link URI is returned. -->
## Accessing multiple links
Redirection to the resource you most likely want is just the default behaviour. The resolver may have multiple links associated with a given identifier and these can be discovered in two ways:
### HTTP-only
You can do an HTTP HEAD request on any GS1 Digital Link URI and suppress your client’s redirection. The resolver will include the full list of available links in its Link response header. **NB** this feature is likely to evolve. The GS1 Digital Link standard suggests (but does not normatively require) use of the [Linkset](https://datatracker.ietf.org/doc/html/draft-ietf-httpapi-linkset-03) draft standard. This *may* one day mean that provision of all available links in the HTTP Link Header is unnecessary (it's known to cause problems if there are a lot of links). At the time of writing, this is an open question. For now though, the following remains true.

The key elements of the HTTP trace for the Dal Giardino example are as follows:

`HTTP/1.1 307 Temporary Redirect`

`Location: https://dalgiardino.com/risotto-rice-with-mushrooms/`

`Link: <https://dalgiardino.com/risotto-rice-with-mushrooms/>; rel="gs1:pip"; type="text/html"; hreflang="en"; title="Product information page", <https://dalgiardino.com/mushroom-squash-risotto/>; rel="gs1:recipeInfo"; type="text/html"; hreflang="en"; title="Recipe website", <https://dalgiardino.com/where-to-buy/>; rel="gs1:hasRetailers"; type="text/html"; hreflang="en"; title="Retailers", <https://dalgiardino.com/about/>; rel="gs1:productSustainabilityInfo"; type="text/html"; hreflang="en"; title="sustainability and recycling"`

The 307 response and the Location header tell you the destination of the redirect that would be applied with a regular GET. The Link header includes a list of options (separated by commas), each of which has a number of fields (separated by semicolons):
1.	The target URL
2.	The link relationship type
3.	The media type of the target resource
4.	The language of the target resource
5.	A human-readable title for the link

For the example response above, the information contained in the Link: response is as follows:

| Target URL | Link relationship type | Media type | Language | Title |
| ---------- | ---------------------- | ---------- | -------- | ----- |
| https://dalgiardino.com/risotto-rice-with-mushrooms | gs1:pip | text/html | en | Product information page |
| https://dalgiardino.com/mushroom-squash-risotto/ | gs1:recipeInfo | text/html | en | Recipe website |
| https://dalgiardino.com/where-to-buy | gs1:hasRetailers | text/html | en | Has retailers |
| https://dalgiardino.com/about/ | gs1:productSustainabilityInfo | text/html | en | Product sustainability information |

### linkType=all
The same list of available links can be obtained as either a JSON object or an HTML page if you use the value ‘all’ for the linkType parameter. For example:

`https://id.gs1.org/01/9506000134352?linkType=all`

The JSON object returned to clients specifying that media type in the HTTP request is identical to the one included in and processed by the HTML page. Resolvers allow the differentiation of links associated with a particular GS1 key according to:
1. The link relationship type (known in GS1 Digital Link as simply the ‘link type’)
2. The language of the target resource
3. The media type of the target resource
4. A further value called context.
The value space (e.g. permitted / defined values) for the context parameter is not defined in the GS1 Digital Link standard but is defined separately for each resolver in the Resolver Description File that can be found as a JSON object at `/.well-known/gs1resolver` for any GS1 conformant resolver (see the [id.gs1.org resolver example](https://id.gs1.org/.well-known/gs1resolver)). The values can be expressed either as an enumerated list (in an array provided as the value of `supportedContextValuesEnumerated` and/or by naming one or more external lists of values as the value of the `supportedContextValuesExternal` property).
The structure of the returned JSON follows the [Linkset](https://datatracker.ietf.org/doc/html/draft-ietf-httpapi-linkset-03) draft standard and can be considered stable. Note the importance of the Accept header when requesting the linkset.

## Requesting a specific link
Rather than redirecting to the default link, a resolver will redirect requests for a specific link type if one is available. For example:

`https://id.gs1.org/01/9506000134352?linkType=gs1:recipeInfo`

will redirect to a recipe for that product rather than the product information page, which is the default. 
GS1 maintains a list of link types as part of its [Web vocabulary](https://mh1.eu/voc/?show=linktypes) but you can get a quick listing of the link types in use in a particular resolver by consulting its [Resolver Description File](https://id.gs1.org/.well-known/gs1resolver). You can get an authoritative list of all the GS1 link types as a JSON object from https://www.gs1.org/voc/?show=linktypes (set the Accept header to application/json for JSON).

## Downloading all available links in bulk
There will soon be a mechanism to access all the links in the GS1 resolver as a bulk download.

## Demo app
A basic app is available that demonstrates the key features that you might want to include in apps of your own. It is for Android devices (only) and is not available through the Play Store so to install it you'll need to [download it from here](https://philarcher.org/temp/GS1ResolverClient.apk) and then override the warning about installing apps from an unnown source. The demo app depends on you having a scanner installed (such as [Barcode scanner](https://play.google.com/store/apps/details?id=com.qrcodescanner.barcodescanner). For emphasis, this is only intended as a demo - GS1 has no plans to create an actual app for use by consumers or business partners (we'll leave that to you). 

# Underlying principles
There are a number of principles that underlie the GD1 Digital Link standard, principles that could readily be applied elsewhere, that is, outside the GS1 identifier system. These are [detailed separately](https://gs1.github.io/DigitalLinkDocs/principles/).

# Where's the community?
Please report any bugs with any of the software by raising an issue in the relevant repository. If you want to ask questions and generally be part of the broader developer community around GS1 Digital Link, please join our [Google group](https://groups.google.com/forum/#!forum/gs1-digital-link-developers)

People are always interested to see what others are working on. Please feel free to [add to the wiki](https://github.com/gs1/DigitalLinkDocs/wiki/GS1-Digital-Link-Use-Cases)!

# Running a pilot
The resolver at `id.gs1.org` is part of the GS1 Registry Platform and operates under strict rules to ensure that any links returned are approved by the brand owner (see the [terms of use](https://www.gs1.org/standards/resolver/terms-of-use)). A separate developers' resolver is available as a sandbox for pilots and tests. GS1 Member Organisations, GCP licensees and solution providers may request an account on the developers' resolver. Contact <a href="mailto:phil.archer@gs1.org">Phil Archer</a> for details. Developers are strongly encouraged to join the [GS1 Digital Link developers' Google group](https://groups.google.com/forum/#!forum/gs1-digital-link-developers)
