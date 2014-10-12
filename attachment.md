# Attachment extension
This document outlines an extension which supports sending and receiving CollectionJson documents containing file attachments. This approach uses a multipart/form-data request / multipart/mixed response as a means of transfering attachments back and forth.

*Note*: Each of the examples below is based on the existing CJ friends [example](http://amundsen.com/media-types/collection/examples/).

It is inspired by this [discussion](https://groups.google.com/forum/#!topic/collectionjson/pzdkNGx-aPE)

The following RFCs form the basis of the approach in this document:

* [1341] (http://www.w3.org/Protocols/rfc1341/7_2_Multipart.html) - The Multipart Content-Type
* [2388] (http://tools.ietf.org/html/rfc2388) - Returning Values from Forms:  multipart/form-data
* [6266] (http://tools.ietf.org/html/rfc6266) - Use of the Content-Disposition Header Field in the Hypertext Transfer Protocol (HTTP)

## Write template
A client may receive a CollectionJson document containing a Write template which accepts attachments which the client can use to send files. 

### Attachment field
A new _attachment_ field is introduced on the data element. For a template, this indicates that this data element is an attachment.

### Example
```javascript
{
  "template" : {
    "data" : [
      {"name" : "full-name", "value" : ""},
      {"name" : "email", "value" : ""},
      {"name" : "avatar", "attachment": ""}
    ]
  }
}
```
## Sending attachments
A client may send a response that contains attachments using the media type "multipart/form-data". It contains the data for the write template as well as attachments.

### Parts
* All _attachment_ fields in the data element must have a corresponding part.
* The part must have a name matching the form element name.

### Example
Below you can can see the request contains a write template with contact information. The template contains an avatar _attachment_ item with the value of the attachment being 'jdoe'. There is an additional part which contains the avatar image which has a _name_ of 'jdoe'
```
content-type: multipart/form-data, boundary=AaB03x

--AaB03x
content-disposition: form-data; name="full-name"
content-type: text/plain; charset=us-ascii
John Doe
--AaB03x
content-disposition: form-data; name="email"
content-type: text/plain; charset=us-ascii
jdoe@example.org
--AaB03x
content-disposition: form-data; name="avatar"
content-type: image/jpeg
...
--AaB03x
```
## Receving attachments
A client may also receive a response that contains attachments. In these cases the response media type will be "multipart/mixed" containing parts for a collection+json document as well as attachments.

### Parts
* The first part in the document will be a CollectionJson document. The document will contain pointers back to the attachments in the response.
* All _attachment_ fields in the data element must have a value set to the name in the content-disposition header of the corresponding part.
* The document part must have a "name" of "document" as it's content-disposition header
* All additional parts will be attachments which relate to the document.
* Each attachment must have a "name" as part of the content-disposition header.

### Attachment Data field
The _attachment_ field in the response indicates that this item has an associated attachment. The _value_ of the attachment matches the _name_ attribute of the Content-Disposition header in one of the parts.

### Example
Below is an example of a response containing attachments. The first part is a document which contains the list of contacts where each contact has an avatar with an _attachment_ field. Then there are additional parts which are the actual attachments. Each attachment's filename corresponds to the value of the _attachment_ field for the item in  the CollectionJson document.

```
content-type: multipart/mixed, boundary=AaB03x
 
--AaB03x
content-type: application/vnd.collection+json
{ "collection" :
  {
    "version" : "1.0",
    "href" : "http://example.org/friends/",
    
    "items" : [
      {
        "href" : "http://example.org/friends/jdoe",
        "data" : [
          {"name" : "full-name", "value" : "John Doe", "prompt" : "Full Name"},
          {"name" : "email", "value" : "jdoe@example.org", "prompt" : "Email"}
          {"name" : "avatar", "attachment" : "jdoe.jpg"}
        ]
      },
      {
        "href" : "http://example.org/friends/mamund",
        "data" : [
          {"name" : "full-name", "value" : "Mike Amundsen", "prompt" : "Full Name"},
          {"name" : "email", "value" : "mca@amundsen.com", "prompt" : "Email"}
          {"name" : "avatar", "attachment" : "mamund.jpg"}
        ]
      }
    }
  }
}
--AaB03x
content-disposition: attachment; filename="jdoe.jpeg"
content-type: image/jpeg
content-transfer-encoding: binary
...
--AaB03x
content-disposition: attachment; filename="mamund.jpeg"
content-type: image/jpeg
content-transfer-Encoding: binary
...
--AaB03x
```

