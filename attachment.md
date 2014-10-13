# Attachment extension
This document outlines an extension which supports sending and receiving collection+json documents containing file attachments. This approach uses a multipart/form-data request / multipart/mixed response as a means of transfering attachments back and forth.

*Note*: Each of the examples below is based on the existing CJ friends [example](http://amundsen.com/media-types/collection/examples/).

It is inspired by this [discussion](https://groups.google.com/forum/#!topic/collectionjson/pzdkNGx-aPE)

The following RFCs form the basis of the approach in this document:

* [1341] (http://www.w3.org/Protocols/rfc1341/7_2_Multipart.html) - The Multipart Content-Type
* [2388] (http://tools.ietf.org/html/rfc2388) - Returning Values from Forms:  multipart/form-data
* [6266] (http://tools.ietf.org/html/rfc6266) - Use of the Content-Disposition Header Field in the Hypertext Transfer Protocol (HTTP)

## Write template
A client MAY receive a CollectionJson document containing a Write template which accepts attachments which the client can use to send files. 

### enc-type field
A new _enc-type_ field is introduced on the template element.
* It MUST have a value of "application/json", "application/vnd.collection+json", or "multipart/form-data".
* For templates with attachments, "multipart/form-data" MUST be used.

### attachment field
A new _attachment_ field is introduced on the data element. 
* This field MUST be set to "true" to indicate to the client that this is an attachment.
** If _enc-type_ is not present / is not set to "multipart/form-data" then clients MUST treat it as text.
** If _attachment_ is set to false, then clients MUST treat it as text.

### Example
```javascript
{
  "template" : {
    "enc-type" : "multipart/form-data",
    "data" : [
      {"name" : "full-name", "value" : ""},
      {"name" : "email", "value" : ""},
      {"name" : "avatar", "attachment": "true"}
    ]
  }
}
```
## Sending attachments
A client MAY send a request that contains attachments using the media type "multipart/form-data". The request contains the data for the write template as well as attachments.

### Parts
* Items with _attachment_ fields in the data element MAY have a corresponding part.
* If there is a corresponding part, then it MUST match the name of a data element in the template.
** If a server receives a request with a part that does not match the data element, it MAY ignore the attachment.

### Example
Below you can can see the request contains a write template with contact information. The template contains an avatar _attachment_ item with the value of the attachment being 'jdoe'. There is an additional part which contains the avatar image which has a _name_ of 'avatar' and a filename of "jdoe.jpg". 
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
content-disposition: form-data; name="avatar"; filename="jdoe.jpg"
content-type: image/jpeg
...
--AaB03x
```
## Receving attachments
A client MAY also receive a response that items which contain attachments. Attachments are indicated by a link with a render value of "attachment". This indicates that href points to a file which SHOULD be downloaded.

### Example
Below is an example of a response containing attachments. The first part is a document which contains the list of contacts where each contact has an avatar with an _attachment_ field. Then there are additional parts which are the actual attachments. Each attachment's filename corresponds to the value of the _attachment_ field for the item in  the collection+json document.

```
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
        ],
        "links": [
          {"name": "avatar", "rel": "related", "render": "attachment", "href":"http://example.org/images/jdoe.jpg"}
        ]
      },
      {
        "href" : "http://example.org/friends/mamund",
        "data" : [
          {"name" : "full-name", "value" : "Mike Amundsen", "prompt" : "Full Name"},
          {"name" : "email", "value" : "mca@amundsen.com", "prompt" : "Email"}
        ],
        "links": [
          {"name": "avatar", "rel": "related", "render": "attachment", "href":"http://example.org/images/mamund.jpg"}
        ]
      }
    }
  }
}
```

**Note** - The server below / gist is currently not up to date with the spec.

To see a gist with real server response with attachments go [here] (https://gist.github.com/glennblock/8db0d1facb69af67af16). You can also hit a live version [here] (http://cj-attachment.azurewebsites.net/friend) with a tool like Fiddler or using curl: `curl http://cj-attachment.azurewebsites.net/friend -v`
