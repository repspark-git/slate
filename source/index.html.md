---
title: RepSpark API Reference

language_tabs:
  - csharp: C#
  - java: Java

toc_footers:
  - <a href='http://www.repspark.com'>Sign Up for a Developer Key</a>

search: true
---

# Introduction

> Comments about code samples look like this.

```csharp
Console.WriteLine("Relevant code samples look like this!");
```

```java
System.out.println("Relevant code samples look like this!");
```

This document describes RepSpark's dynamic API and resources. The design of the API is loosely based on RESTful principles with dynamic extensions to handle the evolving nature of integration challenges.

The APIs are for developers who want to integrate their ERP data with RepSpark's application and for administrators who want to script interactions with RepSpark's servers. The API also allows you to receive transaction logs directly to your inbox or in the HTTP response body.

Because the API is based on open standards, you can use use any HTTP client in any programming language to interact with the API. The API uses several standard HTTP headers as well as several custom HTTP headers. The headers and values are described below.

## Extensibility

Given the number of ERPs and apparel companies, future data format requirements are impossible to predict. As a result, we designed the API with extensibility as a core guiding principle. To implement this ideal, we chose the [JSON schema](#schemas) specification. These schemas provide the ultimate level of flexibility when it comes to data. Unpredictable modifications and extensions are easy to achieve. If your data does not conform to the [generic schemas](#payload-schemas), just let us know and we'll work together to create a custom schema.

## Base URL

All URLs referenced in the documentation have the following base

- [https://api.repspark.net/api](https://api.repspark.net/api)

> How to make an HTTPS request

```csharp
string url = "https://api.repspark.net/api";
HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
```

```java
// For HTTPS to work properly, please use JDK 7 or 8
URL url = new URL("https://api.repspark.net/api");
HttpsURLConnection request = (HttpsURLConnection)url.openConnection();
```

<aside class="notice">
The API is only served over HTTPS to ensure data privacy and security.
</aside>

## HTTP Verbs

HTTP verbs determine how incoming data should be applied to the provided resources.

> How to set an HTTP verb

```csharp
// HttpWebRequest request defined above
request.Method = "POST";
```

```java
// HttpsURLConnection request defined above
request.setRequestMethod("POST");
```

A `GET` request signals the intention to read the data at the destination resource. A `POST` request indicates that the data at the destination resource should be created or updated with the incoming payload.

## HTTP Headers

> How to set an HTTP header

```csharp
// HttpWebRequest request defined above
request.Headers.Add("HeaderName", "HeaderValue");
```

```java
// HttpsURLConnection request defined above
request.setRequestProperty("HeaderName", "HeaderValue");
```

RepSpark's API uses HTTP headers to precisely configure each request.  

Individual headers have detailed descriptions in the following sections.

Headers | &nbsp;
------- | --------
[`Authorization: Basic <BASE64_STRING>`](#credentials) | <span class="label label-warning">REQUIRED</span>
[`X-RepSpark-TransactionToken: <TOKEN_STRING>`](#transaction-token) | <span class="label label-warning">REQUIRED</span>
[`X-RepSpark-SyncMode: <SYNC_MODE>`](#sync-mode) | <span class="label label-warning">REQUIRED</span>
[`X-RepSpark-BatchType: <BATCH_TYPE>`](#batch-type) | <span class="label label-default">optional</span>
[`X-RepSpark-Resource: <RESOURCE_NAME>`](#resources) | <span class="label label-warning">REQUIRED</span>
[`X-RepSpark-SchemaName: <SCHEMA_NAME>`](#schema-name) | <span class="label label-default">optional</span>
[`X-RepSpark-SchemaVersion: <VERSION_STRING>`](#schema-version) | <span class="label label-default">optional</span>
[`X-RepSpark-LogKeywords: <COMMA_SEPARATED_LIST>`](#log-keywords) | <span class="label label-default">optional</span>
[`X-RepSpark-LogLevels: <COMMA_SEPARATED_LIST>`](#log-levels) | <span class="label label-default">optional</span> if `LogKeywords` omitted
[`X-RepSpark-LogSeverities: <COMMA_SEPARATED_LIST>`](#log-severities) | <span class="label label-default">optional</span> if `LogKeywords` omitted
[`X-RepSpark-LogOutputFormat: <FORMAT_STRING>`](#log-format) | <span class="label label-default">optional</span> if `LogKeywords` omitted
[`X-RepSpark-LogContentDisposition: <DISPOSITION_STRING>`](#log-content-dispositions) | <span class="label label-default">optional</span> if `LogKeywords` omitted
[`X-RepSpark-LogDevEmails: <COMMA_SEPARATED_LIST>`](#log-content-dispositions) | <span class="label label-warning">REQUIRED</span> if `LogContentDisposition` set to `Email`

# Credentials

`Authorization: Basic <BASE64_STRING>`

>"HTTP Basic authentication (BA) implementation is the simplest technique for enforcing access controls to web resources because it doesn't require cookies, session identifiers, or login pages; rather, HTTP Basic authentication uses standard fields in the HTTP header, obviating the need for handshakes." -[Basic access authentication](https://en.wikipedia.org/wiki/Basic_access_authentication)

All requests to the API require you to authenticate using HTTP basic authentication to convey your identity. The username is your client key (a 36 character Guid as a string). The password (a 36 character Guid as a string) is your development token. Your client key and dev token are provided by RepSpark.

To make test requests that don't affect live data, use the dummy client key along with any dev token. The dummy client key is `00000000-1111-2222-3333-444444444444`.

The Authorization header is constructed as follows:

1. The username and password are combined into a string: `username:password`

2. The resulting string is then encoded using the [RFC2045-MIME](https://www.ietf.org/rfc/rfc2045.txt) variant of Base64, except not limited to 76 char.

3. The authorization method and a space is then put before the encoded string (e.g. `Basic `).

`Authorization: Basic NDkxZDNkNmUtN….`

> For example, if you use `00000000-1111-2222-3333-444444444444` as the username (i.e. the client key) and `88014bc7-01f2-41d6-8da0-27dca921d601` as the password (i.e. the dev token), then the header is formed as follows:

```csharp
string clientKey = "00000000-1111-2222-3333-444444444444";
string devToken = "88014bc7-01f2-41d6-8da0-27dca921d601";
string usernameAndPassword = clientKey + ":" + devToken;
byte[] bytes = System.Text.Encoding.UTF8.GetBytes(usernameAndPassword);
string base64String = System.Convert.ToBase64String(bytes);
// HttpWebRequest request defined above
request.Headers.Add("Authentication", "Basic " + base64String);
```

```java
String clientKey = "00000000-1111-2222-3333-444444444444";
String devToken = "88014bc7-01f2-41d6-8da0-27dca921d601";
String usernameAndPassword = clientKey + ":" + devToken;
byte[] bytes = usernameAndPassword.getBytes("UTF-8");
// Java 8 only
java.util.Base64.Encoder encoder = java.util.Base64.getEncoder();
String base64String = encoder.encode(bytes);
// HttpsURLConnection request defined above
request.setRequestProperty("Authentication", "Basic " + base64String);
```

<aside class="notice">
Despite the insecure nature of basic authentication, our required use of HTTPS largely mitigates any risk.
</aside>

# Transaction Token

> How to generate a Guid

```csharp
string guid = System.Guid.NewGuid().ToString();
```

```java
String guid = java.util.UUID.randomUUID();
```

`X-RepSpark-TransactionToken: <TOKEN_STRING>`

The API provides transactional functionality to transform large payloads. The transaction token (a 36 character Guid as a string) is an identifier for each sync process. Your application needs to generate this token for each new sync.

For example, if you want to sync a large amount of data (e.g. InvoiceReport), you may need to send multiple calls to the API. All of the calls for this specific sync use the same transaction token.

Next, you may want to sync a small amount of data (e.g. Season). This should only require a single call to the API. Do not use the same transaction token as before. Generate a new Guid to send along with this single request.

# Sync Mode

`X-RepSpark-SyncMode: <SYNC_MODE>`

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

# Batch Type

`X-RepSpark-BatchType: <BATCH_TYPE>`

If no batch type is specified, it performs the entire sync in a single call, executing all of the steps below behind the scenes.

>The Begin, Append, and Commit examples assume that a request exists like the following:

```csharp
string url = "https://api.repspark.net/api";
HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);

request.Method = "POST";

string clientKey = "00000000-1111-2222-3333-444444444444";
string devToken = "88014bc7-01f2-41d6-8da0-27dca921d601";
string usernameAndPassword = clientKey + ":" + devToken;
byte[] bytes = System.Text.Encoding.UTF8.GetBytes(usernameAndPassword);
string base64String = System.Convert.ToBase64String(bytes);
request.Headers.Add("Authentication", "Basic " + base64String);

string transactionToken = System.Guid.NewGuid().ToString();
request.Headers.Add("X-RepSpark-TransactionToken", transactionToken);

request.Headers.Add("X-RepSpark-Resource", "Option");
request.Headers.Add("X-RepSpark-SyncMode", "Delta");
request.Headers.Add("X-RepSpark-LogKeywords", "All");
request.Headers.Add("X-RepSpark-LogLevels", "All");
request.Headers.Add("X-RepSpark-LogFormat", "text/html");
request.Headers.Add("X-RepSpark-LogContentDisposition", "Email");
request.Headers.Add("X-RepSpark-LogDevEmails", "YOUR_EMAIL@YOUR_DOMAIN.TLD");
```

```java
// For HTTPS to work properly, please use JDK 7 or 8
URL url = new URL("https://api.repspark.net/api");
HttpsURLConnection request = (HttpsURLConnection)url.openConnection();

request.setRequestMethod("POST");

String clientKey = "00000000-1111-2222-3333-444444444444";
String devToken = "88014bc7-01f2-41d6-8da0-27dca921d601";
String usernameAndPassword = clientKey + ":" + devToken;
byte[] bytes = usernameAndPassword.getBytes("UTF-8");
// Java 8 only
java.util.Base64.Encoder encoder = java.util.Base64.getEncoder();
String base64String = encoder.encode(bytes);
request.setRequestProperty("Authentication", "Basic " + base64String);

String transactionToken = java.util.UUID.randomUUID();
request.setRequestProperty("X-RepSpark-TransactionToken", transactionToken);

request.setRequestProperty("X-RepSpark-Resource", "Option");
request.setRequestProperty("X-RepSpark-SyncMode", "Delta");
request.setRequestProperty("X-RepSpark-LogKeywords", "All");
request.setRequestProperty("X-RepSpark-LogLevels", "All");
request.setRequestProperty("X-RepSpark-LogFormat", "text/html");
request.setRequestProperty("X-RepSpark-LogContentDisposition", "Email");
request.setRequestProperty("X-RepSpark-LogDevEmails", "YOUR_EMAIL@YOUR_DOMAIN.TLD");
```

Available values: `Begin`, `Append`, `Commit`

The transaction type header sets the progress of the data transfer.

## Begin

```csharp
// HttpWebRequest request defined above
request.Headers.Add("X-RepSpark-BatchType", "Begin");
var response = request.GetResponse();
```

```java
// HttpsURLConnection request defined above
request.setRequestProperty("X-RepSpark-BatchType", "Begin");
Reader response = new InputStreamReader(request.getInputStream());
```

The `Begin` type signals the start of a multi-call data transfer for the given transaction token.

Arguments | &nbsp;
--------- | ------
Data payload | <span class="label label-default">optional</span>

## Append

```csharp
// HttpWebRequest request defined above
request.Headers.Add("X-RepSpark-BatchType", "Append");
var payloadData = @"[
                      {
                        ""Description"": ""Description01"",
                        ""ElementType"": ""ElementType01"",
                        ""KeyCode"": ""KeyCode01"",
                        ""Hidden"": true
                      },
                      {
                        ""Description"": ""Description02"",
                        ""ElementType"": ""ElementType02"",
                        ""KeyCode"": ""KeyCode02""
                      }
                    ]";

using (var requestStream = request.GetRequestStream())
{
    var payloadBytes = System.Text.Encoding.UTF8.GetBytes(payloadData);
    requestStream.Write(payloadBytes, 0, payloadBytes.Length);
}

var response = request.GetResponse();
```

```java
// HttpsURLConnection request defined above
request.setRequestProperty("X-RepSpark-BatchType", "Append");
String payloadData = "[" +
                      "{" +
                        "\"\"Description\"\": \"\"Description01\"\"," +
                        "\"\"ElementType\"\": \"\"ElementType01\"\"," +
                        "\"\"KeyCode\"\": \"\"KeyCode01\"\"," +
                        "\"\"Hidden\"\": true" +
                      "}," +
                      "{" +
                        "\"\"Description\"\": \"\"Description02\"\"," +
                        "\"\"ElementType\"\": \"\"ElementType02\"\"," +
                        "\"\"KeyCode\"\": \"\"KeyCode02\"\"" +
                      "}" +
                    "]";  

request.setDoOutput(true);
DataOutputStream wr = new DataOutputStream(request.getOutputStream());
wr.writeBytes(payloadData);
wr.flush();
wr.close();

Reader response = new InputStreamReader(request.getInputStream());
```

The `Append` type adds the incoming data to the existing data transfer for the given transaction token.

Arguments | &nbsp;
--------- | ------
Data payload | <span class="label label-warning">REQUIRED</span>

## Commit

```csharp
// HttpWebRequest request defined above
request.Headers.Add("X-RepSpark-BatchType", "Commit");
var response = request.GetResponse();
```

```java
// HttpsURLConnection request defined above
request.setRequestProperty("X-RepSpark-BatchType", "Commit");
Reader response = new InputStreamReader(request.getInputStream());
```

The `Commit` type adds the incoming data to the existing data transfer for the given transaction token. Then, it triggers the transformation of all received data to the targeted environment.

Arguments | &nbsp;
--------- | ------
Data payload | <span class="label label-default">optional</span>

## Example Scenario

Let's say you have five years of order reporting data that won't fit in memory. To transfer this data, you have two options.

1. Split the data into many chunks that fit in memory, then make sequential `Append` calls.
2. Stream the data as it is and don't specify batch type.

The first option looks like the following:

1. `Begin` (with or without data)
2. `Append` (with data)
3. `Append` (with data)
4. `Append` (with data)
5. ...
6. `Commit` (with or without data)

The second option looks like the following:

1. Unspecified batch type (stream all data)

In the event that your data is small enough to fit in memory, you may omit batch type with or without streaming.

# Resources

`X-RepSpark-Resource: <RESOURCE_NAME>`

The resources are the type of entities or payloads currently supported. Define the payload name to be accessed in the headers.

The API has the following resources. Details are described in this documentation for accessing or modifying each resource.

## Lookup

Name | Methods
---- | ------
Brand | `POST`
Customer | `POST`
Option | `POST`
Season | `POST`

## Historical Reporting

Name | Methods
---- | ------
InvoiceItemReport | `POST`
InvoiceItemSizeReport | `POST`
InvoiceReport | `POST`
OrderItemReport | `POST`
OrderItemSizeReport | `POST`
OrderReport | `POST`
SalesGoal | `POST`

## Sales

Name | Methods | Query Params
---- | ------ | ------------
Order | `GET` | `StatusId` refers to the status of the order and `OrderId` refers to RepSpark's order ID
OrderConfirmation | `POST`
SimpleOrder | `GET` | `StatusId` refers to the status of the order and `OrderId` refers to RepSpark's order ID

## Stock

Name | Methods
---- | ------
Inventory | `POST`
InventoryLocation | `POST`
Product | `POST`
ProductSizeReference | `POST`
SizeRun | `POST`
Sizing | `POST`
UPC | `POST`

# Schemas

Schemas define the data structure for each payload. They guarantee consistency between the sending and receiving of data.

Depending on your integration status, your implementation may require a specific schema. RepSpark will advise you if this is the case.

JSON schema allows properties to be nullable. When this is the case, the property has a second value of `null`. In addition, `string`s may enforce a maximum length, indicated by the presence of the `maxLength` attribute.

Formatting `date`s as `string`s can be tricky and prone to bugs. As a result, we support the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) standard for formatting `date` and time `string`s. The following are examples of supported date and time `string`s.

- 2015-09-25
- 2015-09-25 15:02:05
- 09/25/2015 15:02:05
- 09/25/2015
- 2012-03-19T07:22Z

Likewise, the `decimal` data type poses serialization challenges as well. All of our `decimal` format `string`s are a combination of the integer-part and fractional-part. By default, the integer-part is 18 and the fractional-part is 2. We write it out like this: `decimal(18,2)`. If the default should not be used, it will be specified in the JSON schema.

To learn more about JSON Schema, please read [Elegant APIs with JSON Schema](https://brandur.org/elegant-apis). The official documentation lives at [http://json-schema.org/](http://json-schema.org/) along with excellent [examples](http://json-schema.org/example1.html).

## Schema Name

`X-RepSpark-SchemaName: <SCHEMA_NAME>`

The schema name specifies the name of the schema for the API call. Unless otherwise noted, it is the name of the entity in question (e.g. Season). If schema name is not specified, it defaults to the destination name.

## Schema Version

`X-RepSpark-SchemaVersion: <VERSION_STRING>`

The schema version specifies the version of the schema. If schema version is not specified, it defaults to 0.1.

# Payload Schemas

<script src="javascripts/jsonformatter.js"></script>
<script type="text/javascript">
document.addEventListener('DOMContentLoaded', function() {
  var elements = document.getElementsByClassName("json-formatter");
  for (var i = 0; i < elements.length; i++) {
    var e = elements.item(i);

    var url = e.dataset["url"] || "";

    (function(e) {
      var xhr = new XMLHttpRequest();
      xhr.open('GET', url, true);
      xhr.setRequestHeader('Content-Type', 'application/json; charset=UTF-8');
      xhr.onload = function() {
        if (xhr.status === 200) {
          e.innerText = "";
          e.appendChild(new JSONFormatter(JSON.parse(xhr.responseText)).render());
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

[Payload schema](../schemas/Customer.schema.json):

<p class="json-formatter" data-url="../schemas/Customer.schema.json">
  Loading...
</p>

## Inventory

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

[Payload schema](../schemas/Inventory.schema.json):

<p class="json-formatter" data-url="../schemas/Inventory.schema.json">
Loading...
</p>

## Inventory Location

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

[Payload schema](../schemas/InventoryLocation.schema.json):

<p class="json-formatter" data-url="../schemas/InventoryLocation.schema.json">
  Loading...
</p>

## Invoice Item Report

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

[Payload schema](../schemas/InvoiceItemReport.schema.json):

<p class="json-formatter" data-url="../schemas/InvoiceItemReport.schema.json">
  Loading...
</p>

## Invoice Report

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

[Payload schema](../schemas/InvoiceReport.schema.json):

<p class="json-formatter" data-url="../schemas/InvoiceReport.schema.json">
  Loading...
</p>

## Option

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

[Payload schema](../schemas/Option.schema.json):

<p class="json-formatter" data-url="../schemas/Option.schema.json">
  Loading...
</p>

## Order

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

[Payload schema](../schemas/Order.schema.json):

<p class="json-formatter" data-url="../schemas/Order.schema.json">
  Loading...
</p>

## Order Confirmation

> Example:

```json
[
  {
    "RepSparkOrderNumber": "RepSparkOrderNumber",
    "ErpOrderNumber": "ErpOrderNumber",
    "ErrorMessage": ""
  }
]
```

[Payload schema](../schemas/OrderConfirmation.schema.json):

<p class="json-formatter" data-url="../schemas/OrderConfirmation.schema.json">
  Loading...
</p>

## Order Item Report

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

[Payload schema](../schemas/OrderItemReport.schema.json):

<p class="json-formatter" data-url="../schemas/OrderItemReport.schema.json">
  Loading...
</p>

## Order Report

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

[Payload schema](../schemas/OrderReport.schema.json):

<p class="json-formatter" data-url="../schemas/OrderReport.schema.json">
  Loading...
</p>

## Product

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

[Payload schema](../schemas/Product.schema.json):

<p class="json-formatter" data-url="../schemas/Product.schema.json">
  Loading...
</p>

## Product Size Reference

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

[Payload schema](../schemas/ProductSizeReference.schema.json):

<p class="json-formatter" data-url="../schemas/ProductSizeReference.schema.json">
  Loading...
</p>

## Sales Goals

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

[Payload schema](../schemas/SalesGoal.schema.json):

<p class="json-formatter" data-url="../schemas/SalesGoal.schema.json">
  Loading...
</p>

## Season

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

[Payload schema](../schemas/Season.schema.json):

<p class="json-formatter" data-url="../schemas/Season.schema.json">
  Loading...
</p>

## Size Run

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

[Payload schema](../schemas/SizeRun.schema.json):

<p class="json-formatter" data-url="../schemas/SizeRun.schema.json">
  Loading...
</p>

## Sizing

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

[Payload schema](../schemas/Sizing.schema.json):

<p class="json-formatter" data-url="../schemas/Sizing.schema.json">
  Loading...
</p>

## UPC

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

[Payload schema](../schemas/UPC.schema.json):

<p class="json-formatter" data-url="../schemas/UPC.schema.json">
  Loading...
</p>

# Logging

We recommend logging API calls and their results to monitor and debug your application. In a production environment, we recommend disabling logging to increase performance.

In the headers below, the value `All` includes every other value.

## Log Keywords

`X-RepSpark-LogKeywords: <COMMA_SEPARATED_LIST>`

Available values: `None`, `Application`, `Validation`, `DataProcessing`, `All`

Log keywords classify the type of log messages. Each specified keyword appears in the generated log messages.

If the keyword header is omitted, the default logging level is `None`.

## Log Levels

`X-RepSpark-LogLevels: <COMMA_SEPARATED_LIST>`

Available values: `None`, `Success`, `Info`, `Warning`, `Debug`, `Error`, `Trace`, `Fatal`, `All`

Log levels classify the severity levels of log messages. Each specified level appears in the generated log messages.

This is the set of available log levels:

- <code class="prettyprint green">Success</code>: A successful operation
- <code class="prettyprint lightblue">Info</code>: Interesting runtime events (e.g. API call received)
- <code class="prettyprint yellow">Warning</code>: A potential problem
- <code class="prettyprint lightgray">Debug</code>: Detailed information on the flow of the system
- <code class="prettyprint pink">Error</code>: A significant problem
- <code class="prettyprint white">Trace</code>: The most detailed information available
- <code class="prettyprint red">Fatal</code>: Program aborted
- `All`: All of the above

## Log Severities

`X-RepSpark-LogSeverities: <COMMA_SEPARATED_LIST>`

Available values: `Low`, `Normal`, `High`

Log severities classify the severity of log messages. Each specified severity appears in the generated log messages.

If the severity header is omitted, the default is `All`.

## Log Format

`X-RepSpark-LogOutputFormat: <FORMAT_STRING>`

Available values: `text/html`, `text/csv`, `application/json`

The log format specifies the output format of log content.

If the log format header is omitted, the default is `text/html`.

## Log Content Dispositions

`X-RepSpark-LogContentDisposition: <DISPOSITION_STRING>`

Available values: `Email`, `Inline`

The log content disposition specifies the method of log delivery.

This is the set of available log content dispositions:

- `Email`: Log output will be emailed to you immediately
- `Inline`: Log output is included in the response body. Not available for `GET` requests.

<aside class="notice">
If the <code>Email</code> option is used, <code>X-RepSpark-LogDevEmails</code> must be set to a comma separated list of email addresses.
</aside>

# Responses

## Response Headers

Header | Description
------ | -----------
[`X-RepSpark-ErrorCode`](#response-codes) | Contains the custom RepSpark error response code
`X-RepSpark-TransactionTrackingNumber` | Helps RepSpark aid in debugging
`X-RepSpark-TransactionBatchTrackingNumber` | Helps RepSpark aid in debugging

## Response Codes

`X-RepSpark-ErrorCode: <REPSPARK_CODE>`

Every request will be followed by a response. Some of these responses have specific messages to aid in debugging. Extra details will appear in place of the `{0}` notation.

RepSpark's API uses the following response codes:

Response Code | RepSpark Code | Message
---------- | -------------- | ------
200 OK | OK | Success
301 Permanently Moved | PermanentlyMoved | Please change HTTP requests to HTTPS requests.
400 Bad Request | MissingRequiredHeader | {0} header was not specified.
400 Bad Request | UnsupportedHeader | HTTP header {0} specified in the request is not supported.
400 Bad Request | UnsupportedQueryParameter | Query parameter {0} specified in the request URI is not supported.
400 Bad Request | InvalidQueryParameter | Query parameter {0} specified in the request URI is not valid.
400 Bad Request | InvalidHttpVerb | The HTTP verb specified was not recognized by the server.
400 Bad Request | InvalidInput | Request input {0} is not valid.
400 Bad Request | InvalidHeaderValue | The value provided for HTTP header {0} was not valid. Verify the value including the format.
400 Bad Request | UnsupportedContentType | 	The specified content type is not supported.
401 Forbidden | MissingAuthenticationInfo | The authentication information was not provided. Verify the value of Authorization header.
401 Forbidden | InvalidAuthenticationInfo | The authentication information was not provided in the correct format. Verify the value of Authorization header.
401 Forbidden | AuthenticationFailed | Server failed to authenticate the request. Make sure the value of the Authorization header is formed correctly.
404 Not Found | ResourceNotFound | The specified resource does not exist.
404 Not Found | SchemaNotFound | The specified schema does not exist.
404 Not Found | ProcessorNotFound | The specified processor does not exist.
404 Not Found | TransactionNotFound | The specified transaction does not exist.
405 Method Not Allowed | UnsupportedHttpVerb | The resource doesn't support the specified HTTP verb.
409 Conflict | TransactionAlreadyExists | Transaction with the same token already exists.
500 Internal Server Error | InternalError | The server encountered an internal error. Please retry the request.
500 Internal Server Error | OperationTimedOut | The operation could not be completed within the permitted time.
500 Internal Server Error | UnexpectedErrorOnPreProcessing | Unexpected error occurred on pre-processing.
500 Internal Server Error | UnexpectedErrorOnProcessing | Unexpected error occurred on processing.
500 Internal Server Error | UnexpectedErrorOnPostProcessing | Unexpected error occurred on post-processing.
503 Service Unavailable | ServerBusy | 	The server is currently unable to receive requests. Please retry your request.
