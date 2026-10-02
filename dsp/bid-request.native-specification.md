# Native Ad Request Markup Details

## 1 Object: Native Markup Request

The Native Object defines the native advertising opportunity available for bid via this bid request. It will be included as a JSON-encoded string in the bid request's imp.native field or as a direct JSON object, depending on the choice of the exchange. While OpenRTB 2.x officially supports only JSON-encoded strings, many exchanges have implemented a formal object. Check with your integration docs.

The Default column dictates how optional parameters should be interpreted if explicit values are not provided.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| ver | string | Optional | "1.2" | Version of the Native Markup version in use. |
| context | integer | Recommended | - | The context in which the ad appears. See Table of Context IDs below for a list of supported context types. |
| contextsubtype | integer | Optional | - | A more detailed context in which the ad appears. See Table of Context SubType IDs below for a list of supported context subtypes. |
| plcmttype | integer | Recommended | - | The design/format/layout of the ad unit being offered. See Table of Placement Type IDs below for a list of supported placement types. |
| plcmtcnt | integer | Optional | 1 | The number of identical placements in this Layout. 1 only |
| seq | integer | Not Supported | 0 | 0 for the first ad, 1 for the second ad, and so on. Note this would generally NOT be used in combination with plcmtcnt - either you are auctioning multiple identical placements (in which case plcmtcnt>1, seq=0) or you are holding separate auctions for distinct items in the feed (in which case plcmtcnt=1, seq=>=1) |
| assets | object array | Required | - | An array of Asset Objects. Any bid response must comply with the array of elements expressed in the bid request. |
| aurlsupport | integer | Not Supported | 0 | Whether the supply source / impression supports returning an assetsurl instead of an asset object. 0 or the absence of the field indicates no such support. |
| durlsupport | integer | Not Supported | 0 | Whether the supply source / impression supports returning a dco url instead of an asset object. 0 or the absence of the field indicates no such support. Beta feature. |
| eventtrackers | object array | Optional | - | Specifies what type of event tracking is supported - see Event Trackers Request Object |
| privacy | integer | Recommended | 0 | Set to 1 when the native ad supports buyer-specific privacy notice. Set to 0 (or field absent) when the native ad doesn't support custom privacy links or if support is unknown. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

**Note:** Prior to VERSION 1.1, the specification could be interpreted as requiring the native request to have a root node with a single field "native" that would contain the object above as its value. The Native Markup Request Object specified above is now the root object.

## 2 Object: Asset

The main container object for each asset requested or supported by Exchange on behalf of the rendering client. Any object that is required is to be flagged as such. Only one of the {title,img,video,data} objects should be present in each object. All others should be null/absent. The id is to be unique within the AssetObject array so that the response can be aligned.

To be more explicit, it is the ID of each asset object that maps the response to the request. So if a request for a title object is sent with id 1, then the response containing the title should have an id of 1.

Since version 1.1 of the spec, there are recommended sizes/lengths/etc with some of the asset types. The goal for asset requirements standardization is to facilitate adoption of native by DSPs by limiting the diverse types/sizes/requirements of assets they must have available to purchase a native ad impression. While great diversity may exist in publishers, advertisers/DSPs can not be expected to provide infinite headline lengths, thumbnail aspect ratios, etc. While we have not gone as far as creating a single standard, we've honed in on a few options that cover the most common cases. SSPs can deviate from these standards, but should understand they may limit applicable DSP demand by doing so. DSPs should feel confident that if they support these standards they'll be able to access most native inventory.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | integer | Required | - | Unique asset ID, assigned by exchange. Typically a counter for the array. |
| required | integer | Optional | 0 | Set to 1 if asset is required (exchange will not accept a bid without it) |
| title | object | Recommended | - | Title object for title assets. See TitleObject definition. |
| img | object | Recommended | - | Image object for image assets. See ImageObject definition. |
| video | object | Not Supported | - | Video object for video assets. See the Video request object definition. Note that in-stream (ie preroll, etc) video ads are not part of Native. Native ads may contain a video as the ad creative itself. |
| data | object | Recommended | - | Data object for brand name, description, ratings, prices etc. See DataObject definition. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 3 Object: Title

The Title object is to be used for title element of the Native ad.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| len | integer | Required | - | Maximum length of the text in the title element. Recommended to be 25, 90, or 140. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 4 Object: Image

The Image object to be used for all image elements of the Native ad such as Icons, Main Image, etc. Recommended sizes and aspect ratios are included in the Image Asset Types section.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| type | integer | Optional | - | Type ID of the image element supported by the publisher. The publisher can display this information in an appropriate format. See Table Image Asset Types. |
| w | integer | Optional | - | Width of the image in pixels. |
| wmin | integer | Recommended | - | The minimum requested width of the image in pixels. This option should be used for any rescaling of images by the client. Either w or wmin should be transmitted. If only w is included, it should be considered an exact requirement. |
| h | integer | Optional | - | Height of the image in pixels. |
| hmin | integer | Recommended | - | The minimum requested height of the image in pixels. This option should be used for any rescaling of images by the client. Either h or hmin should be transmitted. If only h is included, it should be considered an exact requirement. |
| mimes | string array | Optional | All types allowed | Whitelist of content MIME types supported. Popular MIME types include, but are not limited to "image/jpg" "image/gif". Each implementing Exchange should have their own list of supported types in the integration docs. See Wikipedia's MIME page for more information and links to all IETF RFCs. If blank, assume all types are allowed. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 5 Object: Video

The video object to be used for all video elements supported in the Native Ad. This corresponds to the Video object of OpenRTB. Exchange implementers can impose their own specific restrictions. Here are the required attributes of the Video Object. For optional attributes please refer to OpenRTB.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| mimes | string array | Required | - | Content MIME types supported. Popular MIME types include, but are not limited to "video/x-ms-wmv" for Windows Media, and "video/x-flv" for Flash Video, or "video/mp4". Note that native frequently does not support flash. |
| minduration | integer | Required | - | Minimum video ad duration in seconds. |
| maxduration | integer | Required | - | Maximum video ad duration in seconds. |
| protocols | integer array | Required | - | An array of video protocols the publisher can accept in the bid response. See OpenRTB Table 'Video Bid Response Protocols' for a list of possible values. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 6 Object: Data

The Data Object is to be used for all non-core elements of the native unit such as Brand Name, Ratings, Review Count, Stars, Download count, descriptions etc. It is also generic for future native elements not contemplated at the time of the writing of this document. In some cases, additional recommendations are also included in the Data Asset Types table.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| type | integer | Required | - | Type ID of the element supported by the publisher. The publisher can display this information in an appropriate format. See Data Asset Types table for commonly used examples. |
| len | integer | Optional | - | Maximum length of the text in the element's response. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 7 Object: Event Trackers

The event trackers object specifies the types of events the bidder can request to be tracked in the bid response, and which types of tracking are available for each event type, and is included as an array in the request.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| event | integer | Required | - | Type of event available for tracking. See Event Types table. |
| methods | integer array | Required | - | Array of the types of tracking available for the given event. See Event Tracking Methods table. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

### Event Types

| ID | Name | Require/Optional | Description |
|----|------|------------------|-------------|
| 1 | Impression |  | Impression |
| 2 | viewable-mrc50 | Not Supported | Visible impression using MRC definition at 50% in view for 1 second |
| 3 | viewable-mrc100 | Not Supported | 100% in view for 1 second (GroupM standard) |
| 4 | viewable-video50 | Not Supported | Visible impression for video using MRC definition at 50% in view for 2 seconds |

### Event Tracking Methods

| ID | Name | Require/Optional | Description |
|----|------|------------------|-------------|
| 1 | img |  | Image-pixel tracking - URL provided will be inserted as a 1x1 pixel at the time of the event |
| 2 | js | Not Supported |  |

## Sample

### Native
```json
{
  "id": "1-1790905561416-440422-9581-3193763219",
  "at": 1,
  "imp": [
    {
      "id": "42c2b018-c007-4457-b35f-2e60c1782dc5",
      "instl": 0,
      "bidfloor": 0.1947,
      "bidfloorcur": "USD",
      "tagid": "4761011",
      "secure": 1,
      "displaymanager": "admize",
      "displaymanagerver": "1.0.0",
      "native": {
        "ver": "1.2",
        "request": "{\"ver\":\"1.2\",\"assets\":[{\"id\":1,\"required\":1,\"img\":{\"type\":1}},{\"id\":2,\"required\":1,\"img\":{\"type\":3,\"w\":320,\"h\":50}},{\"id\":3,\"required\":1,\"title\":{\"len\":90}},{\"id\":4,\"data\":{\"type\":1}},{\"id\":5,\"data\":{\"type\":2}},{\"id\":6,\"data\":{\"type\":6}},{\"id\":7,\"data\":{\"type\":7}},{\"id\":8,\"required\":1,\"data\":{\"type\":12}}],\"eventtrackers\":[{\"event\":1,\"methods\":[1]}]}"
      }
    }
  ],
  "tmax": 220,
  "cur": ["USD"],
  "badv": ["app.test.com"],
  "bapp": [],
  "bcat": ["IAB25-5", "IAB26-4", "IAB9-9", "IAB25-4", "IAB26-3"],
  "app": {
    "id": "440422",
    "name": "kr.co.ating.atingos",
    "bundle": "kr.co.ating.atingos",
    "storeurl": "kr.co.ating.atingos",
    "publisher": {
      "id": "9bfbbc96-ace0-4cc4-8b6f-3170da3cac26",
      "name": "admize",
      "ext": { "sid": 27 }
    }
  },
  "device": {
    "ua": "Dalvik/2.1.0 (Linux; U; Android 16; SM-F966N Build/BP4A.251205.006)",
    "ip": "106.101.71.221",
    "devicetype": 1,
    "make": "samsung",
    "model": "SM-F966N",
    "os": "Android",
    "osv": "16",
    "language": "ko",
    "ifa": "1c047c7e-4763-4b70-92i8-4c60ae75a0zh",
    "connectiontype": 1,
    "geo": {
      "country": "KOR"
    }
  },
  "regs": { "coppa": 0 },
  "source": {
    "ext": {
      "schain": {
        "ver": "1.0",
        "complete": 1,
        "nodes": [
          {
            "asi": "cauly.net",
            "sid": "7130",
            "rid": "1790905561189363041",
            "hp": 1
          },
          {
            "asi": "admize.io",
            "sid": "p_1875135558794156",
            "rid": "1790905561189363041",
            "hp": 1
          }
        ]
      }
    }
  }
}
```