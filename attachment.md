# Attachment
This document outlines an extension which supports sending and receiving CollectionJson documents containing file attachments. This approach uses a multipart/form-data request/response as a means of transfering attachments back and forth. 

*Note*: Each of the examples below is based on the existing CJ friends [example](http://amundsen.com/media-types/collection/examples/).

It based on this [discussion](https://groups.google.com/forum/#!topic/collectionjson/pzdkNGx-aPE)

## Attachment template
A client may receive a CollectionJson document containing a template which accepts attachments which the client can use to send files. Each attachment will be indicated with a _file_ attribute.

### File data element
A new _file_ attribute is introduced on the data element. For a template, this indicates that this data element is a file.

### Example
```javascript
{
  "template" : {
    "data" : [
      {"name" : "full-name", "value" : ""},
      {"name" : "email", "value" : ""},
      {"name" : "avatar", "file": ""}
    ]
  }
}
```
## Sending attachments
A client may send a response that contains attachments using the media type "multipart/form-data". It contains parts with a CollectionJson template populated with data as well as attachments.

### Parts
* The first part in the document will be a CollectionJson template.
* The document part must have a _name_ of "template" in it's content-disposition header
* All additional parts will be attachments.
* Each attachment may have a "name" as part of the content-disposition header which matches the name in the template.

## File data element
A _file_ element of the template indicates that the client should send an attachment.

### Example
Below you can can see the request contains a write template with contact information. The template contains an avatar _file_ item. There is an additional part which contains the avatar image which has a _name_ of 'avatar'
```
Content-Type: multipart/form-data, boundary=AaB03x
Content-Type: application/vnd.collection+json
--AaB03x
Content-Disposition: form-data; name="template"

{
  "template" : {
    "data" : [
      {"name" : "full-name", "value" : "John Doe"},
      {"name" : "email", "value" : "jdoe@example.org"},
      {"name" : "avatar", "file": ""}
    ]
  }
} 
--AaB03x
Content-Disposition: form-data; name="document"
Content-Type: application/vnd.collection+json

--AaB03x
Content-Disposition: attachment; name="avatar"; filename="johndoe.jpeg"
Content-Type: image/jpeg
Content-Transfer-Encoding: binary
...

--AaB03x
```
## Receving attachments
A client may also receive a response that contains attachments. In these cases the response media type will be "multipart/form-data" containing parts for a CollectionJson document as well as attachments.

### Parts
* The first part in the document will be a CollectionJson document. The document will contain pointers back to the attachments in the response.
* The document part must have a "name" of "document" as it's content-disposition header
* All additional parts will be attachments which relate to the document.
* Each attachment must have a "name" as part of the content-disposition header.

### File Data element
The _file_ element in the response indicates that this item has an associated attachment. The _value_ of the file matches the _name_ attribute of the Content-Disposition header in one of the parts.

### Example
Below is an example of a response containing attachments. The first part is a document which contains the list of contacts where each contact has a _file_ element. Then there are additional parts which are the actual attachments. Each one has a _name_ which the CollectionJson document uses as the key.

```
Content-type: multipart/form-data, boundary=AaB03x
 
--AaB03x
Content-Disposition: form-data; name="document"
Content-Type: application/vnd.collection+json
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
          {"name" : "avatar", "file" : "jdoe"}
        ]
      },
      {
        "href" : "http://example.org/friends/mamund",
        "data" : [
          {"name" : "full-name", "value" : "Mike Amundsen", "prompt" : "Full Name"},
          {"name" : "email", "value" : "mca@amundsen.com", "prompt" : "Email"}
          {"name" : "avatar", "file" : "mamund"}
        ]
      }
    }
  }
}

--AaB03x
Content-Disposition: attachment; name="jdoe"; filename="johndoe.jpeg"
Content-Type: image/jpeg
Content-Transfer-Encoding: binary
...

--AaB03x
Content-Disposition: attachment; name="mamund"; filename="mikeamundsen.jpeg"
Content-Type: image/jpeg
Content-Transfer-Encoding: binary
...

--AaB03x
```
