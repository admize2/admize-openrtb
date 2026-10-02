# Native Ad Response Markup Details

The structure and contents of the Bid Response are the same as in the OpenRTB standard. The difference is in how the ad creative is returned. The native creative shall be returned as a JSON-encoded string in the adm field of the Bid Object. Note some implementers have chosen to use a direct object in a new field rather than JSON encoded string.

## 1 Object: Native Markup Response

The native object is the top level JSON object which identifies a native response. The native object has following attributes:

| Attribute | Type | Require/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| ver | string | Recommended | "1.2" | Version of the Native Markup version in use. |
| assets | object array | Recommended | - | List of native ad's assets. Required if no assetsurl. Recommended as fallback even if assetsurl is provided. |
| assetsurl | string | Not Supported | - | URL of an alternate source for the assets object. The expected response is a JSON object mirroring the assets object in the bid response, subject to certain requirements as specified in the individual objects. Where present, overrides the asset object in the response. |
| dcourl | string | Not Supported | - | URL where a dynamic creative specification may be found for populating this ad, per the Dynamic Content Ads Specification. Note this is a beta option as the interpretation of the Dynamic Content Ads Specification and how to assign those elements into a native ad is outside the scope of this spec and must be agreed offline between the parties or as may be specified in a future revision of the Dynamic Content Ads spec. Where present, overrides the asset object in the response. |
| link | object | Required | - | Destination Link. This is default link object for the ad. Individual assets can also have a link object which applies if the asset is activated(clicked). If the asset doesn't have a link object, the parent link object applies. See LinkObject Definition |
| imptrackers | string array | Optional | - | Array of impression tracking URLs, expected to return a 1x1 image or 204 response - typically only passed when using 3rd party trackers. To be deprecated - replaced with eventtrackers. |
| jstracker | string | Not Supported | - | Optional JavaScript impression tracker. This is a valid HTML, Javascript is already wrapped in `<script>` tags. It should be executed at impression time where it can be supported. To be deprecated - replaced with eventtrackers. |
| eventtrackers | object array | Optional | - | Array of tracking objects to run with the ad, in response to the declared supported methods in the request. Replaces imptrackers and jstracker, to be deprecated. **(only supports event tracking method type .img)** |
| privacy | string | Optional | - | If support was indicated in the request, URL of a page informing the user about the buyer's targeting activity. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 2 Object: Asset

Corresponds to the Asset Object in the request. The main container object for each asset requested or supported by Exchange on behalf of the rendering client. Any object that is required is to be flagged as such. Only one of the {title,img,video,data} objects should be present in each object. All others should be null/absent. The id is to be unique within the AssetObject array so that the response can be aligned.

| Attribute | Type | Require/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | integer | Optional | - | Optional if assetsurl/dcourl is being used; required if embedded asset is being used. |
| required | integer | Optional | 0 | Set to 1 if asset is required. (bidder requires it to be displayed). |
| title | object | Optional | - | Title object for title assets. See TitleObject definition. |
| img | object | Optional | - | Image object for image assets. See ImageObject definition. |
| video | object | Not Supported | - | Video object for video assets. See Video response object definition. Note that in-stream video ads are not part of Native. Native ads may contain a video as the ad creative itself. |
| data | object | Optional | - | Data object for ratings, prices etc. |
| link | object | Optional | - | Link object for call to actions. The link object applies if the asset item is activated (clicked). If there is no link object on the asset, the parent link object on the bid response applies. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 3 Object: Title

Corresponds to the Title Object in the request, with the value filled in.

If using assetsurl or dcourl response rather than embedded asset response, it is recommended that three title objects be provided, the length of each of which is less than or equal to the three recommended maximum title lengths (25,90,140).

| Attribute | Type | Require/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| text | string | Required | - | The text associated with the text element. |
| len | integer | Optional | - | The length of the title being provided. Required if using assetsurl/dcourl representation, optional if using embedded asset representation. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 4 Object: Image 

Corresponds to the Image Object in the request. The Image object to be used for all image elements of the Native ad such as Icons, Main Image, etc.

It is recommended that if assetsurl/dcourl is being used rather than embedded assets, that an image of each recommended aspect ratio (per the Image Types table) be provided for image type 3.

| Attribute | Type | Require/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| type | integer | Optional | - | Required for assetsurl or dcourl responses, not required for embedded asset responses. The type of image element being submitted from the Image Asset Types table. |
| url | string | Required | - | URL of the image asset. |
| w | integer | Recommended | - | Width of the image in pixels. Recommended for embedded asset responses. Required for assetsurl/dcourl responses if multiple assets of same type submitted. |
| h | integer | Recommended | - | Height of the image in pixels. Recommended for embedded asset responses. Required for assetsurl/dcourl responses if multiple assets of same type submitted. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 5 Object: Data

Corresponds to the Data Object in the request, with the value filled in. The Data Object is to be used for all miscellaneous elements of the native unit such as Brand Name, Ratings, Review Count, Stars, Downloads, Price count etc. It is also generic for future native elements not contemplated at the time of the writing of this document.

| Attribute | Type | Require/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| type | integer | Optional | - | Required for assetsurl/dcourl responses, not required for embedded asset responses. The type of data element being submitted from the Data Asset Types table. |
| len | integer | Optional | - | Required for assetsurl/dcourl responses, not required for embedded asset responses. The length of the data element being submitted. Where applicable, must comply with the recommended maximum lengths in the Data Asset Types table. |
| value | string | Required | - | The formatted string of data to be displayed. Can contain a formatted value such as "5 stars" or "$10" or "3.4 stars out of 5". |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 6 Object: Video

Corresponds to the Video Object in the request, yet containing a value of a conforming VAST tag as a value.

| Attribute | Type | Require/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| vasttag | string | Required | - | vast xml. |

## 7 Object: Link

Used for 'call to action' assets, or other links from the Native ad. This Object should be associated to its peer object in the parent Asset Object or as the master link in the top level Native Ad response object. When that peer object is activated (clicked) the action should take the user to the location of the link.

| Attribute | Type | Require/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| url | string | Required | - | Landing URL of the clickable link. |
| clicktrackers | string array | Optional | - | List of third-party tracker URLs to be fired on click of the URL. |
| fallback | string (URL) | Optional | - | Fallback URL for deeplink. To be used if the URL given in url is not supported by the device. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## 8 Object: Event Tracker

The event trackers response is an array of objects and specifies the types of events the bidder wishes to track and the URLs/information to track them. Bidder must only respond with methods indicated as available in the request. Note that most javascript trackers expect to be loaded at impression time, so it's not generally recommended for the buyer to respond with javascript trackers on other events, but the appropriateness of this is up to each buyer.

| Attribute | Type | Require/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| event | integer | Required | - | Type of event to track. See Event Types table. |
| method | integer | Required | - | Type of tracking requested. See Event Tracking Methods table. |
| url | string | Optional | - | The URL of the image or js. Required for image or js, optional for custom. |
| customdata | object containing key:value pairs | Optional | - | To be agreed individually with the exchange, an array of key:value objects for custom tracking, for example the account number of the DSP with a tracking company. IE {"accountnumber":"123"}. |
| ext | object | Optional | - | This object is a placeholder that may contain custom JSON agreed to by the parties to support flexibility beyond the standard defined in this specification |

## Reference & Enumeration

### Context Type IDs

| ID | Description |
|----|-------------|
| 1 | Content-centric context such as newsfeed, article, image gallery, video gallery, or similar |
| 2 | Social-centric context such as social network feed, email, chat, or similar |
| 3 | Product context such as product listings, details, recommendations, reviews, or similar |

### Context Sub Type IDs

| ID | Description |
|----|-------------|
| 10 | General or mixed content |
| 11 | Primarily article content |
| 12 | Primarily video content |
| 13 | Primarily audio content |
| 14 | Primarily image content |
| 15 | User-generated content - forums, comments, etc. |
| 20 | General social content such as a general social network |
| 21 | Primarily email content |
| 22 | Primarily chat/IM content |
| 30 | Content focused on selling products, whether digital or physical |
| 31 | Application store/marketplace |
| 32 | Product reviews site primarily (which may sell product secondarily) |

### Placement Type IDs

| ID | Description |
|----|-------------|
| 1 | In the feed of content |
| 2 | In the atomic unit of the content |
| 3 | Outside the core content |
| 4 | Recommendation widget |

### Image Asset Types

| ID | Name | Description |
|----|------|-------------|
| 1 | Icon | Icon Image |
| 3 | Main | Ad Image |

### Data Asset Types

| ID | Name |
|----|------|
| 1 | sponsored |
| 2 | desc |
| 3 | rating |
| 4 | likes |
| 5 | downloads |
| 6 | price |
| 7 | saleprice |
| 8 | phone |
| 9 | address |
| 10 | desc2 |
| 11 | displayurl |
| 12 | ctatext |

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
  "id": "4b6436abc2e54e0869e1d39c2ee4c432",
  "bidid": "b-4b6436abc2e5-0001",
  "cur": "USD",
  "seatbid": [
    {
      "seat": "seat-1",
      "bid": [
        {
          "id": "bid-16abc7ca-0001",
          "impid": "16abc7ca-0f13-4ae1-6a6e-15ffabec0650",
          "price": 0.15,
          "nurl": "https://win.dsp.example.com/win?bid=bid-16abc7ca-0001&price=${AUCTION_PRICE}",
          "lurl": "https://win.dsp.example.com/loss?bid=bid-16abc7ca-0001&reason=${AUCTION_LOSS}",
          "adm": "{\"ver\":\"1.2\",\"assets\":[{\"id\":1,\"data\":{\"value\":\"더 알아보기\"}},{\"id\":2,\"title\":{\"text\":\"테스트-타이틀\"}},{\"id\":3,\"img\":{\"url\":\"https://cdn.dsp.example.com/icon/80x80.jpg\",\"w\":80,\"h\":80}},{\"id\":4,\"img\":{\"url\":\"https://cdn.dsp.example.com/main/1200x627.jpg\",\"w\":1200,\"h\":627}}],\"link\":{\"url\":\"https://www.example.com/landing\",\"clicktrackers\":[\"https://track.dsp.example.com/click?bid=bid-16abc7ca-0001\"]},\"imptrackers\":[\"https://track.dsp.example.com/imp?bid=bid-16abc7ca-0001&price=${AUCTION_PRICE}\"]}",
          "adid": "ad-1001",
          "adomain": ["example.com"],
          "cid": "cmp-2001",
          "crid": "cr-3001",
          "cat": ["IAB9"]
        }
      ]
    }
  ]
}
```