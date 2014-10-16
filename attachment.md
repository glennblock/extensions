# Attachment extension
This extension outlines an extension which supports sending and receiving `collection+json` documents  containing file attachments. This approach uses a multipart/form-data request for sending attachments and annotated links in the response for surfacing attachments to be downloaded.

*Note*: Each of the examples below is based on the existing CJ friends [example](http://amundsen.com/media-types/collection/examples/).

It is inspired by this [discussion](https://groups.google.com/forum/#!topic/collectionjson/pzdkNGx-aPE)

The following RFCs/documents form the basis of the approach in this document:

* [1341] (http://www.w3.org/Protocols/rfc1341/7_2_Multipart.html) - The Multipart Content-Type
* [2388] (http://tools.ietf.org/html/rfc2388) - Returning Values from Forms:  multipart/form-data
* [draft-multi-part-form-data] (https://tools.ietf.org/html/draft-ietf-appsawg-multipart-form-data-05) - Updates to RFC2388
* [6266] (http://tools.ietf.org/html/rfc6266) - Use of the Content-Disposition Header Field in the Hypertext Transfer Protocol (HTTP)
* 

## Live example
To see a response containing attachment links, use the following command or just open in a browser: 

```text
curl http://cj-attachment.azurewebsites.net/friends -v
```

You can also grab a specific friend, using the href for the item in the payload. Also you can just as well create a direct request using the shortname (first initial + last name)

```test
curl http://cj-attachment.azurewebsites.net/friends/mamundsen
```

To send a multipart/form-data request with files, use the following command subsituting `thumbnail.png` with your own image.

```text
curl -0 -v -include --form full-name="John Doe" --form email="jdoe@example.org" --form blog="http://example.org/jdoe" --form avatar="@./thumbnail.png" http://cj-attachment.azurewebsites.net/friends
```

After the image is uploaded, you will find it at `http://cj-attachment.azurewebsites.net/avatars?name={file}` i.e. `http://cj-attachment.azurewebsites.net/avatars/name=thumbnail.png` in the previous case. You can open it in a browser and it will download.

## Write template
A client MAY receive a CollectionJson document containing a Write template which accepts attachments which the client can use to send files. 

### Content-type property
This extension defines a new optional property for the template object: `contentType`. The two valid values for `contentType ` are:

* `multipart/form-data` (this is the one to use for uploading attachments)
* `application/vnd.collection+json` (this is the one to use for sending regular CJ documents) If the content-type property is missing, clients SHOULD use `application/vnd.collection+json` when sending a CJ document. If the content-type property is not supported and/or the provided value is not understood by the client, the client MUST use `application/vnd.collection+json` when sending CJ documents.

### Attachment property
This extension defines a new property for the data object: `attachment`. This property is only valid for data objects that are children of the template object. 

The two valid values for the `attachment` property:

* true (treat this data element as an attachment to be uploaded)
* false (treat this data element as a text element) If the client does not support the `attachment` property and/or the value of this property is not understood, the client MUST treat the data element as a text element.

### Example
Below you can can see the request contains a friend write template which specifies a content-type of `multipart/form-data`. The `avatar` data object is marked as an attachment, indicating that a file should be uploaded.

```javascript
{
  "template" : {
    "contentType" : "multipart/form-data",
    "data" : [
      {"name" : "full-name", "value" : ""},
      {"name" : "email", "value" : ""},
      {"name" : "avatar", "attachment": "true"}
    ]
  }
}
```
## Sending attachments
A client MAY send a request that contains attachments using the media type "multipart/form-data". The request contains the data for the write template passed as form data via the `content-disposition header` as well as file uploads.

### Mulipart requests
This extension compiles with with [RFC2388] and [draft-multi-part-form-data] (https://tools.ietf.org/html/draft-ietf-appsawg-multipart-form-data-05) for sending requests. You should refer to these documents for the guidelines. 

**Note**: At this time, multi-file attachements per template data object are not supported.

### Example
Below you can can see the request contains three body parts. The first two contain textual information for full-name and email, while the third is an attachment containing the avatar.  
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
A client MAY receive a response which contains links which represent downloadable attachments.

## Attachment render value
A new `render` value of `attachment` is introduced for links. This informs the client that it should treat the `href` for the link as downloadable.  Clients that do not support the `attachment` value for render MUST treat the associated href as a navigation link.

### Example
Below is an example of a response containing links which are attachments, namely the `avatar` link has a `render` value of `attachment`. The client in this casse SHOULD download the associated image.

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

