---
title: RepSpark API Reference

language_tabs:
  - csharp: C#

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

> Comments about code samples look like this.

```csharp
Console.WriteLine("Relevant code samples look like this!");
```

This document describes RepSpark's dynamic API and resources. The design of the API is loosely based on RESTful principals with dynamic extensions to handle the evolving nature of integration challenges.

The APIs are for developers who want to integrate their ERP data with RepSpark's application and for administrators who want to script interactions with RepSpark's servers. The API also allows you to receive transaction logs directly to your inbox or in the HTTP response body.

Because the API is based on open standards, you can use use any HTTP client in any programming language to interact with the API. The API uses several standard HTTP headers as well as several custom HTTP headers. The headers and values are described below.

## Base URL

All URLs referenced in the documentation have the following base

- [https://api.repspark.net/api](https://api.repspark.net/api)

> How to make an HTTPS request

```csharp
String url = "https://api.repspark.net/api";
HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
```

<aside class="notice">
The API is only served over HTTPS to ensure data privacy and security.
</aside>

# Credentials

`Authorization: Basic <BASE64STRING>`

All requests to the API require you to authenticate using HTTP basic authentication to convey your identity. The username is your client key (a 36 character Guid as a string). The password (a 36 character Guid as a string) is your dev token. Your client key and dev token are provided by RepSpark.

>"HTTP Basic authentication (BA) implementation is the simplest technique for enforcing access controls to web resources because it doesn't require cookies, session identifiers, or login pages; rather, HTTP Basic authentication uses standard fields in the HTTP header, obviating the need for handshakes." -[Basic access authentication](https://en.wikipedia.org/wiki/Basic_access_authentication)

The Authorization header is constructed as follows:

1. The username and password are combined into a string as follows:

`username:password`

2. The resulting string is then encoded using the [RFC2045-MIME](https://www.ietf.org/rfc/rfc2045.txt) variant of Base64, except not limited to 76 char.

3. The authorization method and a space is then put before the encoded string (e.g. `Basic `).

> For example, if you use `491d3d6e-7f0a-472d-bde5-b01b61d1f57e` as the username (i.e. the client key) and `88014bc7-01f2-41d6-8da0-27dca921d601` as the password (i.e. the dev token), then the header is formed as follows:

```csharp
String clientKey = "491d3d6e-7f0a-472d-bde5-b01b61d1f57e";
String devToken = "88014bc7-01f2-41d6-8da0-27dca921d601";
String usernameAndPassword = clientKey + ":" + devToken;
byte[] bytes = System.Text.Encoding.UTF8.GetBytes(usernameAndPassword);
String base64String = System.Convert.ToBase64String(bytes);
String authHeader = "Authorization: Basic " + base64String;
```

`Authorization: Basic NDkxZDNkNmUtN….`

Despite the insecure nature of basic authentication, our required use of HTTPS largely mitigates any risk.

# Transaction Token

> How to generate a Guid

```csharp
String guid = System.Guid.NewGuid().ToString();
```

`X-RepSpark-TransactionToken: <TOKENSTRING>`

The API provides transactional functionality to transform large payloads. The transaction token (a 36 character Guid as a string) is an identifier for each sync process. Your application needs to generate this token for each new sync.

For example, if you want to sync a large amount of data (e.g. InvoiceReport), you may need to send multiple calls to the API. All of the calls for this specific sync use the same transaction token.

Next, you may want to sync a small amount of data (e.g. Season). This should only require a single call to the API. Do not use the same transaction token as before. Generate a new Guid to send along with this single request.

# Sync Mode

`X-RepSpark-SyncMode: <SYNCMODE>`

Available values: `Full`, `FullByType`, `Delta`

The sync mode header sets the scope of the data to be sent.

## Full

The `Full` SyncMode replaces all of the existing data with the incoming data.

## FullByType

The `FullByType` SyncMode replaces all of the existing data of the specified type with the incoming data. The API currently supports the following:

- ProductType for the Product resource
- ElementType for the Option resource

## Delta

The `Delta` SyncMode updates existing data and inserts new data intelligently instead of overwriting everything.

# Transaction Type

`X-RepSpark-TransactionType: <TRANSACTIONTYPE>`

Available values: `Begin`, `Append`, `Commit`, `Atomic`

The transaction type header sets the progress of the data transfer.

## Begin

The `Begin` type signals the start of a multi-call data transfer for the given transaction token. The data payload is optional.

## Append

The `Append` type adds the incoming data to the existing data transfer for the given transaction token. The data payload is required.

## Commit

The `Commit` type adds the incoming data to the existing data transfer for the given transaction token. Then, it triggers the transformation of all received data to the targeted environment. The data payload is optional.

## Atomic

The `Atomic` type exists apart from the previous three types. Do not use it when using `Begin`, `Append`, and `Commit`.

An `Atomic` transaction performs the entire sync in a single call, executing all of the steps above behind the scenes.

## Example

Let's say you have five years of order reporting data that won't fit in memory. To transfer this data, you have two options.

1. Split the data into many chunks that fit in memory, then make sequential `Append` calls.
2. Stream the data as it is with the `Atomic` transaction type.

The first option looks like the following:

1. `Begin` (with or without data)
2. `Append` (with data)
3. `Append` (with data)
4. `Append` (with data)
5. ...
6. `Commit` (with or without data)

The second option looks like the following:

1. `Atomic` (stream all data)

In the event that your data is small enough to fit in memory, you may use the `Atomic` transaction type with or without streaming.

# Resources

`X-RepSpark-Destination: <RESOURCENAME>`

The resources are the type of entities or payloads currently supported. Define the payload name to be accessed in the headers.

The API has the following sub resources. Details are described in this documentation for accessing or modifying each resource.

## Lookup

- Brand
- Customer
- Option
- Season

## Historical Reporting

- InvoiceItemReport
- InvoiceItemSizeReport
- InvoiceReport
- OrderItemReport
- OrderItemSizeReportSalesGoal
- OrderReport
- PricingMinimum

## Sales

- Order
- OrderConfirmation
- SimpleOrder

## Stock

- Inventory
- InventoryLocation
- Product
- ProductSizeReference
- SizeRun
- Sizing
- SKU
- UPC

# Schemas

Schemas define the data structure for each payload. They guarantee consistency between the sending and receiving of data.

Depending on your integration status, your implementation may require a specific schema. RepSpark will advise you if this is the case.

To learn more about JSON Schema, please read [Elegant APIs with JSON Schema](https://brandur.org/elegant-apis).

## Schema Name

`X-RepSpark-SchemaName: <SCHEMANAME>`

The schema name specifies the name of the schema for the API call. Unless otherwise noted, it is the name of the entity in question (e.g. Season). If schema name is not specified, it defaults to the destination name.

## Schema Version

`X-RepSpark-SchemaVersion: <VERSIONFLOAT>`

The schema version specifies the version of the schema. If schema version is not specified, it defaults to 0.1.

# Processors

RepSpark provides components for data processing to customize data and also increase the stability and reliability of data transformation. It also allows us to improve test-driven development of our API.

Depending on your integration status, your implementation may require a specific processor. RepSpark will advise you if this is the case.

## Processor Name

`X-RepSpark-ProcessorName: Api<RESOURCE>Processor`

Example values: `ApiCustomerProcessor`, `ApiOptionProcessor`, `ApiSeasonProcessor`

The processor name specifies the name of the processor for the API call.

## Processor Version

`X-RepSpark-ProcessorVersion: 0.1`

The processor version specifies the version of the processor. If processor version is not specified, it defaults to 0.1.

# Payload Schemas

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.2.0/styles/railscasts.min.css">

<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.2.0/highlight.min.js" ></script>

<script type="text/javascript">
document.addEventListener('DOMContentLoaded', function() {
  var elements = document.getElementsByClassName("highlight-json");
  for (var i = 0; i < elements.length; i++) {
    var e = elements.item(i);

    var url = e.dataset["url"] || "";

    (function(e) {
      var xhr = new XMLHttpRequest();
      xhr.open('GET', url, true);
      xhr.setRequestHeader('Content-Type', 'application/json; charset=UTF-8');
      xhr.onload = function() {
        if (xhr.status === 200) {
          e.innerText = xhr.responseText;
          hljs.highlightBlock(e);    
        } else {
          e.innerText = "Failed to load schema.";
        }
      };

      xhr.send();
    })(e);
  }
});
</script>

## Customer

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/Customer.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "CustomerCode": "CustomerCode1",
    "Address1": "Address1",
    "City": "City01",
    "State": "State01",
    "Zip": "Zip01",
    "Name": "Name1",
    "Enabled": true
  },
  {
    "CustomerCode": "CustomerCode2",
    "Address1": "Address2",
    "City": "City02",
    "State": "State02",
    "Zip": "Zip02",
    "Name": "Name2"
  }
]
```

## Inventory

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/Inventory.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "ProductNumber": "ProductNumber01",
    "SizeCode": "SizeCode01",
    "AvailableQuantity": "1",
    "AvailableDate": "10/10/2015"
  },
  {
    "ProductNumber": "ProductNumber02",
    "SizeCode": "SizeCode02",
    "AvailableQuantity": "2",
    "AvailableDate": "10/10/2015"
  }
]
```

## Inventory Location

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/InventoryLocation.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "LocationCode": "LocationCode01",
    "LocationDescription": "LocationDescription01",
    "Active": true
  },
  {
    "BrandCode": "LocationCode02",
    "DivisionCode": "LocationDescription02"
  }
]
```

## Invoice Item Report

> Payload Schema:

<pre>
  <code class="highlight-json" data-url="../schemas/InvoiceItemReport.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "InvoiceNumber": "InvoiceNumber01",
    "InvoiceLineNumber": "InvoiceLineNumber01",
    "ProductNumber": "ProductNumber01",
    "ColorCode": "ColorCode01",
    "ColorDescription": "ColorDescription01"
  },
  {
    "InvoiceNumber": "InvoiceNumber02",
    "InvoiceLineNumber": "InvoiceLineNumber02",
    "ProductNumber": "ProductNumber02",
    "ColorCode": "ColorCode02",
    "ColorDescription": "ColorDescription02"
  }
]
```

## Invoice Report

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/InvoiceReport.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "InvoiceNumber": "InvoiceNumber01",
    "BillingCustomerCode": "BillingCustomerCode01",
    "ShippingCustomerCode": "ShippingCustomerCode01",
    "ShippingMethodDescription": "ShippingMethodDescription01",
    "InvoiceCreatedDate": "09/09/2015",
    "InvoiceSentDate": "09/09/2015",
    "FreightAmount": "100.00",
    "InvoiceAmount": "100.00",
    "ShippingMethodCode": "ShippingMethodCode01"
  },
  {
    "InvoiceNumber": "InvoiceNumber02",
    "BillingCustomerCode": "BillingCustomerCode02",
    "ShippingCustomerCode": "ShippingCustomerCode02",
    "ShippingMethodDescription": "ShippingMethodDescription02",
    "InvoiceCreatedDate": "09/09/2015",
    "InvoiceSentDate": "09/09/2015",
    "FreightAmount": "100.00",
    "InvoiceAmount": "100.00",
    "ShippingMethodCode": "ShippingMethodCode02"
  }
]
```

## Option

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/Option.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "Description": "Description01",
    "ElementType": "ElementType01",
    "KeyCode": "KeyCode01",
    "Hidden": true
  },
  {
    "Description": "Description02",
    "ElementType": "ElementType02",
    "KeyCode": "KeyCode02"
  }
]
```

## Order

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/Order.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "OrderId": "OrderId01",
    "OrderGuid": "OrderGuid01",
    "LogOrderNumber": "LogOrderNumber01",
    "ConfirmationCode": "ConfirmationCode01",
    "UserName": "UserName01",
    "BrandCode": "BrandCode01",
    "SeasonCode": "SeasonCode01",
    "DivisionCode": "DivisionCode01",
    "CustomerCode": "CustomerCode01",
    "StoreCode": "StoreCode01",
    "SalesPersonCode": "SalesPersonCode01",
    "PurchaseOrder": "PurchaseOrder01",
    "EntryDate": "10/10/2015",
    "OrderDate": "10/10/2015",
    "StartDate": "10/10/2015",
    "CancelDate": "10/10/2015",
    "InHandDate": "10/10/2015",
    "LastUpdated": "10/10/2015",
    "LastUpdatedBy": "LastUpdatedBy01",
    "TypeCode": "TypeCode01",
    "Comments": "Comments01",
    "TermsCode": "TermsCode01",
    "ShipViaCode": "ShipViaCode01",
    "SpecialHandling": "SpecialHandling01",
    "DropShipName": "DropShipName01",
    "DropShipAttn": "DropShipAttn01",
    "DropShipAddress1": "DropShipAddress101",
    "DropShipAddress2": "DropShipAddress201",
    "DropShipCity": "DropShipCity01",
    "DropShipState": "DropShipState01",
    "DropShipZip": "DropShipZip01",
    "DropShipCountry": "DropShipCountry01",
    "TotalUnits": "10",
    "TotalAmount": "100.00",
    "LocationCode": "LocationCode01",
    "CurrencyCode": "CurrencyCode01",
    "PriceModification": "PriceModification01",
    "PricingPlanId": "PricingPlanId01",
    "PricingTierCode": "PricingTierCode01",
    "IgnoresPricingPlans": "IgnoresPricingPlans01",
    "UsesBuyingGroup": "UsesBuyingGroup01",
    "CatalogCode": "CatalogCode01",
    "StatusId": "StatusId01",
    "SourceId": "SourceId01",
    "OrderItems": "OrderItems01",
    "Extensions": "Extensions01"
  }
]
```

## Order Item Report

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/OrderItemReport.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "OrderNumber": "OrderNumber01",
    "LineNumber": "LineNumber01",
    "ProductNumber": "ProductNumber01",
    "ProductName": "ProductName01"
  },
  {
    "OrderNumber": "OrderNumber02",
    "LineNumber": "LineNumber01",
    "ProductNumber": "ProductNumber02",
    "ProductName": "ProductName02"
  }
]
```

## Order Report

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/OrderReport.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "BrandCode": "BrandCode01",
    "OrderNumber": "OrderNumber01",
    "ShippingCustomerCode": "ShippingCustomerCode01"
  },
  {
    "BrandCode": "BrandCode02",
    "OrderNumber": "OrderNumber02",
    "ShippingCustomerCode": "ShippingCustomerCode02"
  }
]
```

## Product

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/Product.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "ProductNumber": "ProductNumber01",
    "ProductName": "ProductName01",
    "SizeScaleCode": "SizeScaleCode01",
    "WholesalePrice": "20.00",
    "Enabled": true
  },
  {
    "ProductNumber": "ProductNumber02",
    "ProductName": "ProductName02",
    "SizeScaleCode": "SizeScaleCode02",
    "WholesalePrice": "20.00"
  }
]
```

## Product Size Reference

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/ProductSizeReference.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "ProductNumber": "ProductNumber1",
    "SizeCode": "SizeCode01",
    "GenderCode": "GenderCode01"
  },
  {
    "ProductNumber": "ProductNumber2",
    "SizeCode": "SizeCode02"
  }
]
```

## Sales Goals

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/SalesGoal.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "OrderTypeCode": "OrderTypeCode01",
    "UserCode": "UserCode01"
  },
  {
    "OrderTypeCode": "OrderTypeCode02",
    "UserCode": "UserCode02"
  }
]
```

## Season

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/Season.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "SeasonCode": "FALL15",
    "Description": "Fall 2015",
    "Enabled": true
  },
  {
    "SeasonCode": "SUMMER15",
    "Description": "Summer 2015"
  }
]
```

## Size Run

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/SizeRun.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "SizeRunCode": "SizeRunCode01",
    "Sizes": "Sizes01"
  },
  {
    "SizeRunCode": "SizeRunCode02",
    "Sizes": "Sizes02"
  }
]
```

## Sizing

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/Sizing.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "SizeCode": "SizeCode01",
    "SizeScaleCode": "SizeScaleCode01",
    "Enabled": true
  },
  {
    "SizeCode": "SizeCode02",
    "SizeScaleCode": "SizeScaleCode02"
  }
]
```

## UPC

> Payload schema:

<pre>
  <code class="highlight-json" data-url="../schemas/UPC.schema.json">
    Loading...
  </code>
</pre>

> Example:

```json
[
  {
    "ProductNumber": "ProductNumber01",
    "SizeCode": "SizeCode01",
    "UPC": "UPC01"
  },
  {
    "ProductNumber": "ProductNumber02",
    "SizeCode": "SizeCode02",
    "UPC": "UPC02"
  }
]
```

# Logging

We recommend logging API calls and their results to monitor and debug your application. In a production environment, we recommend disabling logging to increase performance.

In the headers below, the value `All` includes every other value.

## Log Keywords

`X-RepSpark-LogKeywords: <COMMA SEPARATED LIST>`

Available values: `None`, `Application`, `Validation`, `DataProcessing`, `All`

Log keywords classify the type of log messages. Each specified keyword appears in the generated log messages.

If the keyword header is omitted, the default logging level is `None`.

## Log Levels

`X-RepSpark-LogLevels: <COMMA SEPARATED LIST>`

Available values: `None`, `Success`, `Info`, `Warning`, `Debug`, `Error`, `Trace`, `Fatal`, `All`

Log levels classify the severity levels of log messages. Each specified level appears in the generated log messages.

This is the set of available log levels:

- `Success`: A successful operation
- `Info`: Interesting runtime events (e.g. API call received)
- `Warning`: A potential problem
- `Debug`: Detailed information on the flow of the system
- `Error`: A significant problem
- `Trace`: The most detailed information available
- `Fatal`: Program aborted
- `All`: All of the above

## Log Severities

`X-RepSpark-LogSeverities: <COMMA SEPARATED LIST>`

Available values: `Low`, `Normal`, `High`

Log severities classify the severity of log messages. Each specified severity appears in the generated log messages.

## Log Format

`X-RepSpark-LogOutputFormat: <FORMATSTRING>`

Available values: `text/plain`, `text/html`, `text/csv`, `application/json`

The log format specifies the output format of log content.

## Log Content Dispositions

`X-RepSpark-LogContentDisposition: <DISPOSITIONSTRING>`

Available values: `Email`, `Inline`

The log content disposition specifies the method of log delivery.

This is the set of available log content dispositions:

- `Email`: Log output will be emailed to you immediately
- `Inline`: Log output is included in the response body

<aside class="notice">
If the <code>Email</code> option is used, <code>X-RepSpark-LogDevEmails</code> must be set to a comma separated list of email addresses.
</aside>
