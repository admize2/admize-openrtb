# Bid Request

This page describes the bid request format sent from SSPs to Admize.

## Request Header

HTTP headers sent with each bid request:

- `x-openrtb-version`: 2.3, 2.5
- `Accept-Encoding`: gzip
- `Content-Type`: application/json
- `Content-Encoding`: gzip (only for DSPs who support gzip compression)
- `Content-Length`: [request body length]

## 1 Object: BidRequest

The top-level bid request object contains a globally unique bid request or auction ID. This `id` attribute is required as is at least one impression object (Section 4). Other attributes in this top-level object establish rules and restrictions that apply to all impressions being offered.

There are also several subordinate objects that provide detailed data to potential buyers. Among these are the Site and App objects, which describe the type of published media in which the impression(s) appear. These objects are highly recommended, but only one applies to a given bid request depending on whether the media is browser-based web content or a non-browser application, respectively.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Required | - | Unique ID of the bid request, provided by the exchange. |
| imp | object array | Required | - | Array of Imp objects (Section 4) representing the impressions offered. At least 1 Imp object is required. |
| site | object | Optional. At least one of site or app must be present. | - | Details via a Site object (Section 13) about the publisher's website. Only applicable and recommended for websites. |
| app | object | Optional. At least one of site or app must be present. | - | Details via an App object (Section 14) about the publisher's app (i.e., non-browser applications). Only applicable and recommended for apps. |
| device | object | Required | - | Details via a Device object (Section 18) about the user's device to which the impression will be delivered. |
| user | object | Recommended | - | Details via a User object (Section 20) about the human user of the device; the advertising audience. |
| test | integer | Optional | 0 | Indicator of test mode in which auctions are not billable, where 0 = live mode, 1 = test mode. |
| at | integer | Optional | 2 | Auction type, where 1 = First Price, 2 = Second Price Plus. Exchange-specific auction types can be defined using values greater than 500. |
| tmax | integer | Optional | - | Maximum time in milliseconds the exchange allows for bids to be received including Internet latency to avoid timeout. This value supersedes any a priori guidance from the exchange. |
| wseat | string array | Optional | - | White list of buyer seats (e.g., advertisers, agencies) allowed to bid on this impression. IDs of seats and knowledge of the buyer's customers to which they refer must be coordinated between bidders and the exchange a priori. At most, only one of wseat and bseat should be used in the same request. Omission of both implies no seat restrictions. |
| bseat | string array | Optional | - | Block list of buyer seats (e.g., advertisers, agencies) restricted from bidding on this impression. IDs of seats and knowledge of the buyer's customers to which they refer must be coordinated between bidders and the exchange a priori. At most, only one of wseat and bseat should be used in the same request. Omission of both implies no seat restrictions. |
| allimps | integer | Optional | 0 | Flag to indicate if Exchange can verify that the impressions offered represent all of the impressions available in context (e.g., all on the web page, all video spots such as pre/mid/post roll) to support road-blocking. 0 = no or unknown, 1 = yes, the impressions offered represent all that are available. |
| cur | string array | Optional | - | Array of allowed currencies for bids on this bid request using ISO-4217 alpha codes. Recommended only if the exchange accepts multiple currencies. |
| wlang | string array | Optional | - | White list of languages for creatives using ISO-639-1-alpha-2. Omission implies no specific restrictions, but buyers would be advised to consider language attribute in the Device and/or Content objects if available. |
| bcat | string array | Optional | - | Blocked advertiser categories using the IAB content categories. Refer to List 5.1. |
| badv | string array | Optional | - | Block list of advertisers by their domains (e.g., "ford.com"). |
| bapp | string array | Optional | - | Block list of applications by their platform-specific exchange-independent application identifiers. On Android, these should be bundle or package names (e.g., com.foo.mygame). On iOS, these are numeric IDs. |
| source | object | Optional | - | A Source object (Section 2) that provides data about the inventory source and which entity makes the final decision. |
| regs | object | Optional | - | A Regs object (Section 3) that specifies any industry, legal, or governmental regulations in force for this request. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 2 Object: Source

This object describes the nature and behavior of the entity that is the source of the bid request upstream from the exchange. The primary purpose of this object is to define post-auction or upstream decisioning when the exchange itself does not control the final decision. A common example of this is header bidding, but it can also apply to upstream server entities such as another RTB exchange, a mediation platform, or an ad server combines direct campaigns with 3rd party demand in decisioning.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| fd | integer | Recommended | - | Entity responsible for the final impression sale decision, where 0 = exchange, 1 = upstream source. |
| tid | string | Recommended | - | Transaction ID that must be common across all participants in this bid request (e.g., potentially multiple exchanges). |
| pchain | string | Recommended | - | Payment ID chain string containing embedded syntax described in the TAG Payment ID Protocol v1.0. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 2-1 Object: Source Ext
| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| schain | object | Optional | - | Admize schain Object (Refer to [OpenRTB SupplyChain object](https://github.com/InteractiveAdvertisingBureau/openrtb/blob/main/supplychainobject.md)) |

### 2-1-1 Supply Chain (schain) Example

```json
{
  "source": {
    "ext": {
      "schain": {
        "ver": "1.0",
        "complete": 1,
        "nodes": [
          {
            "asi": "cauly.net",
            "sid": "6374",
            "hp": 1
          },
          {
            "asi": "center.admize.io",
            "sid": "p_1875135558794156",
            "hp": 1
          }
        ]
      }
    }
  }
}
```

## 3 Object: Regs

This object contains any legal, governmental, or industry regulations that apply to the request. The coppa flag signals whether or not the request falls under the United States Federal Trade Commission's regulations for the United States Children's Online Privacy Protection Act ("COPPA").

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| coppa | integer | Optional | - | Flag indicating if this request is subject to the COPPA regulations established by the USA FTC, where 0 = no, 1 = yes. Refer to Section 7.5 for more information. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 4 Object: Imp

This object describes an ad placement or impression being auctioned. A single bid request can include multiple Imp objects, a use case for which might be an exchange that supports selling all ad positions on a given page. Each Imp object has a required ID so that bids can reference them individually.

The presence of Banner (Section 6), Video (Section 7), and/or Native (Section 9) objects subordinate to the Imp object indicates the type of impression being offered. The publisher can choose one such type which is the typical case or mix them at their discretion. However, any given bid for the impression must conform to one of the offered types.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Required | - | A unique identifier for this impression within the context of the bid request (typically, starts with 1 and increments. |
| metric | object array | Optional | - | An array of Metric object (Section 5). |
| banner | object | Optional. At least one of banner, video, or native must be present. | - | A Banner object (Section 6); required if this impression is offered as a banner ad opportunity. |
| video | object | Optional. At least one of banner, video, or native must be present. | - | A Video object (Section 7); required if this impression is offered as a video ad opportunity. |
| audio | object | Not Supported | - | An Audio object (Section 8); required if this impression is offered as an audio ad opportunity. |
| native | object | Optional. At least one of banner, video, or native must be present. | - | A Native object (Section 9); required if this impression is offered as a native ad opportunity. |
| pmp | object | Not Supported | - | A Pmp object (Section 11) containing any private marketplace deals in effect for this impression. |
| displaymanager | string | Optional | Admize | Name of ad mediation partner, SDK technology, or player responsible for rendering ad (typically video or mobile). Used by some ad servers to customize ad code by partner. Recommended for video and/or apps. |
| displaymanagerver | string | Optional | 1.0.0 | Version of ad mediation partner, SDK technology, or player responsible for rendering ad (typically video or mobile). Used by some ad servers to customize ad code by partner. Recommended for video and/or apps. |
| instl | integer | Optional | 0 | 1 = the ad is interstitial or full screen, 0 = not interstitial. |
| tagid | string | Optional | - | Identifier for specific ad placement or ad tag that was used to initiate the auction. This can be useful for debugging of any issues, or for optimization by the buyer. |
| bidfloor | float | Optional | 0 | Minimum bid for this impression expressed in CPM. |
| bidfloorcur | string | Optional | "USD" | Currency specified using ISO-4217 alpha codes. This may be different from bid currency returned by bidder if this is allowed by the exchange. |
| clickbrowser | integer | Optional | - | Indicates the type of browser opened upon clicking the creative in an app, where 0 = embedded, 1 = native. Note that the Safari View Controller in iOS 9.x devices is considered a native browser for purposes of this attribute. |
| secure | integer | Optional | - | Flag to indicate if the impression requires secure HTTPS URL creative assets and markup, where 0 = non-secure, 1 = secure. If omitted, the secure state is unknown, but non-secure HTTP support can be assumed. |
| iframebuster | string array | Optional | - | Array of exchange-specific names of supported iframe busters. |
| exp | integer | Optional | - | Advisory as to the number of seconds that may elapse between the auction and the actual impression. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 5 Object: Metric

This object is associated with an impression as an array of metrics. These metrics can offer insight into the impression to assist with decisioning such as average recent viewability, click-through rate, etc. Each metric is identified by its type, reports the value of the metric, and optionally identifies the source or vendor measuring the value.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| type | string | Required | - | Type of metric being presented using exchange curated string names which should be published to bidders a priori. |
| value | float | Required | - | Number representing the value of the metric. Probabilities must be in the range 0.0 – 1.0. |
| vendor | string | Recommended | - | Source of the value using exchange curated string names which should be published to bidders a priori. If the exchange itself is the source versus a third party, "EXCHANGE" is recommended. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 6 Object: Banner

This object represents the most general type of impression. Although the term "banner" may have very specific meaning in other contexts, here it can be many things including a simple static image, an expandable ad unit, or even in-banner video (refer to the Video object in Section 7 for the more generalized and full featured video ad units). An array of Banner objects can also appear within the Video to describe optional companion ads defined in the VAST specification.

The presence of a Banner as a subordinate of the Imp object indicates that this impression is offered as a banner type impression. At the publisher's discretion, that same impression may also be offered as video, audio, and/or native by also including as Imp subordinates objects of those types. However, any given bid for the impression must conform to one of the offered types.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| format | object array | Recommended | - | Array of format objects (Section 10) representing the banner sizes permitted. If none are specified, then use of the h and w attributes is highly recommended. |
| w | integer | Required | - | Exact width in device independent pixels (DIPS); recommended if no format objects are specified. |
| h | integer | Required | - | Exact height in device independent pixels (DIPS); recommended if no format objects are specified. |
| btype | integer array | Optional | - | Blocked banner ad types. Refer to List 5.2. |
| battr | integer array | Optional | - | Blocked creative attributes. Refer to List 5.3. |
| pos | integer | Optional | - | Ad position on screen. Refer to List 5.4. |
| mimes | string array | Optional | - | Content MIME types supported. Popular MIME types may include "application/x-shockwave-flash", "image/jpg", and "image/gif". |
| topframe | integer | Optional | - | Indicates if the banner is in the top frame as opposed to an iframe, where 0 = no, 1 = yes. |
| expdir | integer array | Optional | - | Directions in which the banner may expand. Refer to List 5.5. |
| api | integer array | Optional | - | List of supported API frameworks for this impression. Refer to List 5.6. If an API is not explicitly listed, it is assumed not to be supported. |
| id | string | Optional | - | Unique identifier for this banner object. Recommended when Banner objects are used with a Video object (Section 7) to represent an array of companion ads. Values usually start at 1 and increase with each object; should be unique within an impression. |
| vcm | integer | Optional | - | Relevant only for Banner objects used with a Video object (Section 7) in an array of companion ads. Indicates the companion banner rendering mode relative to the associated video, where 0 = concurrent, 1 = end-card. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 7 Object: Video

This object represents an in-stream video impression. Many of the fields are non-essential for minimally viable transactions, but are included to offer fine control when needed. Video in OpenRTB generally assumes compliance with the VAST standard. As such, the notion of companion ads is supported by optionally including an array of Banner objects (refer to the Banner object in Section 6) that define these companion ads.

The presence of a Video as a subordinate of the Imp object indicates that this impression is offered as a video type impression. At the publisher's discretion, that same impression may also be offered as banner, audio, and/or native by also including as Imp subordinates objects of those types. However, any given bid for the impression must conform to one of the offered types.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| mimes | string array | Required | - | Content MIME types supported (e.g., "video/x-ms-wmv", "video/mp4"). |
| minduration | integer | Required | - | Minimum video ad duration in seconds. |
| maxduration | integer | Required | - | Maximum video ad duration in seconds. |
| protocols | integer array | Required | - | Array of supported video protocols. Refer to List 5.8. At least one supported protocol must be specified in either the protocol or protocols attribute. **Supports VAST Only (VAST 2.0 - 4.0)** |
| w | integer | Recommended | - | Width of the video player in device independent pixels (DIPS). |
| h | integer | Recommended | - | Height of the video player in device independent pixels (DIPS). |
| startdelay | integer | Recommended | - | Indicates the start delay in seconds for pre-roll, mid-roll, or post-roll ad placements. Refer to List 5.12 for additional generic values. |
| placement | integer | Optional | - | Placement type for the impression. Refer to List 5.9. |
| linearity | integer | Optional | - | Indicates if the impression must be linear, nonlinear, etc. If none specified, assume all are allowed. Refer to List 5.7. |
| skip | integer | Optional | - | Indicates if the player will allow the video to be skipped, where 0 = no, 1 = yes. If a bidder sends markup/creative that is itself skippable, the Bid object should include the attr array with an element of 16 indicating skippable video. Refer to List 5.3. |
| skipmin | integer | Optional | 0 | Videos of total duration greater than this number of seconds can be skippable; only applicable if the ad is skippable. |
| skipafter | integer | Optional | 0 | Number of seconds a video must play before skipping is enabled; only applicable if the ad is skippable. |
| sequence | integer | Optional | - | If multiple ad impressions are offered in the same bid request, the sequence number will allow for the coordinated delivery of multiple creatives. |
| battr | integer array | Optional | - | Blocked creative attributes. Refer to List 5.3. |
| maxextended | integer | Optional | - | Maximum extended ad duration if extension is allowed. If blank or 0, extension is not allowed. If -1, extension is allowed, and there is no time limit imposed. If greater than 0, then the value represents the number of seconds of extended play supported beyond the maxduration value. |
| minbitrate | integer | Optional | - | Minimum bit rate in Kbps. |
| maxbitrate | integer | Optional | - | Maximum bit rate in Kbps. |
| boxingallowed | integer | Optional | 1 | Indicates if letter-boxing of 4:3 content into a 16:9 window is allowed, where 0 = no, 1 = yes. |
| playbackmethod | integer array | Optional | - | Playback methods that may be in use. If none are specified, any method may be used. Refer to List 5.10. Only one method is typically used in practice. As a result, this array may be converted to an integer in a future version of the specification. It is strongly advised to use only the first element of this array in preparation for this change. |
| playbackend | integer | Optional | - | The event that causes playback to end. Refer to List 5.11. |
| delivery | integer array | Optional | - | Supported delivery methods (e.g., streaming, progressive). If none specified, assume all are supported. Refer to List 5.15. |
| pos | integer | Optional | - | Ad position on screen. Refer to List 5.4. |
| companionad | object array | Optional | - | Array of Banner objects (Section 6) if companion ads are available. |
| api | integer array | Optional | - | List of supported API frameworks for this impression. Refer to List 5.6. If an API is not explicitly listed, it is assumed not to be supported. |
| companiontype | integer array | Optional | - | Supported VAST companion ad types. Refer to List 5.14. Recommended if companion Banner objects are included via the companionad array. If one of these banners will be rendered as an end-card, this can be specified using the vcm attribute with the particular banner (Section 6). |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 7-1 Object: Video.Ext

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| rewarded | integer | Required | - | Non Rewarded = 0, Rewarded = 1. |

## 8 Object: Audio

This object represents an audio type impression. Many of the fields are non-essential for minimally viable transactions, but are included to offer fine control when needed. Audio in OpenRTB generally assumes compliance with the DAAST standard. As such, the notion of companion ads is supported by optionally including an array of Banner objects (refer to the Banner object in Section 6) that define these companion ads.

The presence of a Audio as a subordinate of the Imp object indicates that this impression is offered as an audio type impression. At the publisher's discretion, that same impression may also be offered as banner, video, and/or native by also including as Imp subordinates objects of those types. However, any given bid for the impression must conform to one of the offered types.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| mimes | string array | Required | - | Content MIME types supported (e.g., "audio/mp4"). |
| minduration | integer | Recommended | - | Minimum audio ad duration in seconds. |
| maxduration | integer | Recommended | - | Maximum audio ad duration in seconds. |
| protocols | integer array | Recommended | - | Array of supported audio protocols. Refer to List 5.8. |
| startdelay | integer | Recommended | - | Indicates the start delay in seconds for pre-roll, mid-roll, or post-roll ad placements. Refer to List 5.12. |
| sequence | integer | Optional | - | If multiple ad impressions are offered in the same bid request, the sequence number will allow for the coordinated delivery of multiple creatives. |
| battr | integer array | Optional | - | Blocked creative attributes. Refer to List 5.3. |
| maxextended | integer | Optional | - | Maximum extended ad duration if extension is allowed. If blank or 0, extension is not allowed. If -1, extension is allowed, and there is no time limit imposed. If greater than 0, then the value represents the number of seconds of extended play supported beyond the maxduration value. |
| minbitrate | integer | Optional | - | Minimum bit rate in Kbps. |
| maxbitrate | integer | Optional | - | Maximum bit rate in Kbps. |
| delivery | integer array | Optional | - | Supported delivery methods (e.g., streaming, progressive). If none specified, assume all are supported. Refer to List 5.15. |
| companionad | object array | Optional | - | Array of Banner objects (Section 6) if companion ads are available. |
| api | integer array | Optional | - | List of supported API frameworks for this impression. Refer to List 5.6. If an API is not explicitly listed, it is assumed not to be supported. |
| companiontype | integer array | Optional | - | Supported DAAST companion ad types. Refer to List 5.14. Recommended if companion Banner objects are included via the companionad array. |
| maxseq | integer | Optional | - | The maximum number of ads that can be played in an ad pod. |
| feed | integer | Optional | - | Type of audio feed. Refer to List 5.16. |
| stitched | integer | Optional | - | Indicates if the ad is stitched with audio content or delivered independently, where 0 = no, 1 = yes. |
| nvol | integer | Optional | - | Volume normalization mode. Refer to List 5.17. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 9 Object: Native

This object represents a native type impression. Native ad units are intended to blend seamlessly into the surrounding content (e.g., a sponsored Twitter or Facebook post). As such, the response must be well-structured to afford the publisher fine-grained control over rendering.

The Native Subcommittee has developed a companion specification to OpenRTB called the Dynamic Native Ads API. It defines the request parameters and response markup structure of native ad units. This object provides the means of transporting request parameters as an opaque string so that the specific parameters can evolve separately under the auspices of the Dynamic Native Ads API. Similarly, the ad markup served will be structured according to that specification.

The presence of a Native as a subordinate of the Imp object indicates that this impression is offered as a native type impression. At the publisher's discretion, that same impression may also be offered as banner, video, and/or audio by also including as Imp subordinates objects of those types. However, any given bid for the impression must conform to one of the offered types.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| request | string | Required | - | Request payload complying with the Native Ad Specification. |
| ver | string | Recommended | - | Version of the Dynamic Native Ads API to which request complies; highly recommended for efficient parsing. |
| api | integer array | Optional | - | List of supported API frameworks for this impression. Refer to List 5.6. If an API is not explicitly listed, it is assumed not to be supported. |
| battr | integer array | Optional | - | Blocked creative attributes. Refer to List 5.3. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 10 Object: Format

This object represents an allowed size (i.e., height and width combination) or Flex Ad parameters for a banner impression. These are typically used in an array where multiple sizes are permitted. It is recommended that either the w/h pair or the wratio/hratio/wmin set (i.e., for Flex Ads) be specified.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| w | integer | Optional | - | Width in device independent pixels (DIPS). |
| h | integer | Optional | - | Height in device independent pixels (DIPS). |
| wratio | integer | Optional | - | Relative width when expressing size as a ratio. |
| hratio | integer | Optional | - | Relative height when expressing size as a ratio. |
| wmin | integer | Optional | - | The minimum width in device independent pixels (DIPS) at which the ad will be displayed the size is expressed as a ratio. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 11 Object: Pmp

This object is the private marketplace container for direct deals between buyers and sellers that may pertain to this impression. The actual deals are represented as a collection of Deal objects. Refer to Section 7.3 for more details.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| private_auction | integer | Optional | 0 | Indicator of auction eligibility to seats named in the Direct Deals object, where 0 = all bids are accepted, 1 = bids are restricted to the deals specified and the terms thereof. |
| deals | object array | Optional | - | Array of Deal (Section 12) objects that convey the specific deals applicable to this impression. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 12 Object: Deal

This object constitutes a specific deal that was struck a priori between a buyer and a seller. Its presence with the Pmp collection indicates that this impression is available under the terms of that deal. Refer to Section 7.3 for more details.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Required | - | A unique identifier for the direct deal. |
| bidfloor | float | Optional | 0 | Minimum bid for this impression expressed in CPM. |
| bidfloorcur | string | Optional | "USD" | Currency specified using ISO-4217 alpha codes. This may be different from bid currency returned by bidder if this is allowed by the exchange. |
| at | integer | Optional | - | Optional override of the overall auction type of the bid request, where 1 = First Price, 2 = Second Price Plus, 3 = the value passed in bidfloor is the agreed upon deal price. Additional auction types can be defined by the exchange. |
| wseat | string array | Optional | - | Whitelist of buyer seats (e.g., advertisers, agencies) allowed to bid on this deal. IDs of seats and the buyer's customers to which they refer must be coordinated between bidders and the exchange a priori. Omission implies no seat restrictions. |
| wadomain | string array | Optional | - | Array of advertiser domains (e.g., advertiser.com) allowed to bid on this deal. Omission implies no advertiser restrictions. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 13 Object: Site

This object should be included if the ad supported content is a website as opposed to a non-browser application. A bid request must not contain both a Site and an App object. At a minimum, it is useful to provide a site ID or page URL, but this is not strictly required.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Required | - | Exchange-specific site ID. |
| name | string | Required | - | Site name (may be aliased at the publisher's request). |
| domain | string | Required | - | Domain of the site (e.g., "mysite.foo.com"). |
| cat | string array | Optional | - | Array of IAB content categories of the site. Refer to List 5.1. |
| sectioncat | string array | Optional | - | Array of IAB content categories that describe the current section of the site. Refer to List 5.1. |
| pagecat | string array | Optional | - | Array of IAB content categories that describe the current page or view of the site. Refer to List 5.1. |
| page | string | Optional | - | URL of the page where the impression will be shown. |
| ref | string | Optional | - | Referrer URL that caused navigation to the current page. |
| search | string | Optional | - | Search string that caused navigation to the current page. |
| mobile | integer | Optional | - | Indicates if the site has been programmed to optimize layout when viewed on mobile devices, where 0 = no, 1 = yes. |
| privacypolicy | integer | Optional | - | Indicates if the site has a privacy policy, where 0 = no, 1 = yes. |
| publisher | object | Optional | - | Details about the Publisher (Section 15) of the site. |
| content | object | Optional | - | Details about the Content (Section 16) within the site. |
| keywords | string | Optional | - | Comma separated list of keywords about the site. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 14 Object: App

This object should be included if the ad supported content is a non-browser application (typically in mobile) as opposed to a website. A bid request must not contain both an App and a Site object. At a minimum, it is useful to provide an App ID or bundle, but this is not strictly required.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Required | - | Exchange-specific app ID. |
| name | string | Required | - | App name (may be aliased at the publisher's request). |
| bundle | string | Required | - | A platform-specific application identifier intended to be unique to the app and independent of the exchange. On Android, this should be a bundle or package name (e.g., com.foo.mygame). On iOS, it is typically a numeric ID. |
| domain | string | Optional | - | Domain of the app (e.g., "mygame.foo.com"). |
| storeurl | string | Optional | - | App store URL for an installed app; for IQG 2.1 compliance. |
| cat | string array | Optional | - | Array of IAB content categories of the app. Refer to List 5.1. |
| sectioncat | string array | Optional | - | Array of IAB content categories that describe the current section of the app. Refer to List 5.1. |
| pagecat | string array | Optional | - | Array of IAB content categories that describe the current page or view of the app. Refer to List 5.1. |
| ver | string | Optional | - | Application version. |
| privacypolicy | integer | Optional | - | Indicates if the app has a privacy policy, where 0 = no, 1 = yes. |
| paid | integer | Optional | - | 0 = app is free, 1 = the app is a paid version. |
| publisher | object | Optional | - | Details about the Publisher (Section 15) of the app. |
| content | object | Optional | - | Details about the Content (Section 16) within the app. |
| keywords | string | Optional | - | Comma separated list of keywords about the app. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 15 Object: Publisher

This object describes the publisher of the media in which the ad will be displayed. The publisher is typically the seller in an OpenRTB transaction.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Optional | - | Exchange-specific publisher ID. |
| name | string | Optional | - | Publisher name (may be aliased at the publisher's request). |
| cat | string array | Optional | - | Array of IAB content categories that describe the publisher. Refer to List 5.1. |
| domain | string | Optional | - | Highest level domain of the publisher (e.g., "publisher.com"). |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 16 Object: Content

This object describes the content in which the impression will appear, which may be syndicated or non-syndicated content. This object may be useful when syndicated content contains impressions and does not necessarily match the publisher's general content. The exchange might or might not have knowledge of the page where the content is running, as a result of the syndication method. For example might be a video impression embedded in an iframe on an unknown web property or device.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Optional | - | ID uniquely identifying the content. |
| episode | integer | Optional | - | Episode number. |
| title | string | Optional | - | Content title. Video Examples: "Search Committee" (television), "A New Hope" (movie), or "Endgame" (made for web). Non-Video Example: "Why an Antarctic Glacier Is Melting So Quickly" (Time magazine article). |
| series | string | Optional | - | Content series. Video Examples: "The Office" (television), "Star Wars" (movie), or "Arby 'N' The Chief" (made for web). Non-Video Example: "Ecocentric" (Time Magazine blog). |
| season | string | Optional | - | Content season (e.g., "Season 3"). |
| artist | string | Optional | - | Artist credited with the content. |
| genre | string | Optional | - | Genre that best describes the content (e.g., rock, pop, etc). |
| album | string | Optional | - | Album to which the content belongs; typically for audio. |
| isrc | string | Optional | - | International Standard Recording Code conforming to ISO-3901. |
| producer | object | Optional | - | Details about the content Producer (Section 17). |
| url | string | Optional | - | URL of the content, for buy-side contextualization or review. |
| cat | string array | Optional | - | Array of IAB content categories that describe the content producer. Refer to List 5.1. |
| prodq | integer | Optional | - | Production quality. Refer to List 5.13. |
| context | integer | Optional | - | Type of content (game, video, text, etc.). Refer to List 5.18. |
| contentrating | string | Optional | - | Content rating (e.g., MPAA). |
| userrating | string | Optional | - | User rating of the content (e.g., number of stars, likes, etc.). |
| qagmediarating | integer | Optional | - | Media rating per IQG guidelines. Refer to List 5.19. |
| keywords | string | Optional | - | Comma separated list of keywords describing the content. |
| livestream | integer | Optional | - | 0 = not live, 1 = content is live (e.g., stream, live blog). |
| sourcerelationship | integer | Optional | - | 0 = indirect, 1 = direct. |
| len | integer | Optional | - | Length of content in seconds; appropriate for video or audio. |
| language | string | Optional | - | Content language using ISO-639-1-alpha-2. |
| embeddable | integer | Optional | - | Indicator of whether or not the content is embeddable (e.g., an embeddable video player), where 0 = no, 1 = yes. |
| data | object array | Optional | - | Additional content data. Each Data object (Section 21) represents a different data source. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 17 Object: Producer

This object defines the producer of the content in which the ad will be shown. This is particularly useful when the content is syndicated and may be distributed through different publishers and thus when the producer and publisher are not necessarily the same entity.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Optional | - | Content producer or originator ID. Useful if content is syndicated and may be posted on a site using embed tags. |
| name | string | Optional | - | Content producer or originator name (e.g., "Warner Bros"). |
| cat | string array | Optional | - | Array of IAB content categories that describe the content producer. Refer to List 5.1. |
| domain | string | Optional | - | Highest level domain of the content producer (e.g., "producer.com"). |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 18 Object: Device

This object provides information pertaining to the device through which the user is interacting. Device information includes its hardware, platform, location, and carrier data. The device can refer to a mobile handset, a desktop computer, set top box, or other digital device.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| ua | string | Recommended | - | Browser user agent string. |
| geo | object | Recommended | - | Location of the device assumed to be the user's current location defined by a Geo object (Section 19). |
| dnt | integer | Recommended | - | Standard "Do Not Track" flag as set in the header by the browser, where 0 = tracking is unrestricted, 1 = do not track. |
| lmt | integer | Recommended | - | "Limit Ad Tracking" signal commercially endorsed (e.g., iOS, Android), where 0 = tracking is unrestricted, 1 = tracking must be limited per commercial guidelines. |
| ip | string | Recommended | - | IPv4 address closest to device. |
| ipv6 | string | Optional | - | IP address closest to device as IPv6. |
| devicetype | integer | Optional | - | The general type of device. Refer to List 5.21. |
| make | string | Optional | - | Device make (e.g., "Apple"). |
| model | string | Optional | - | Device model (e.g., "iPhone"). |
| os | string | Required | - | Device operating system (e.g., "iOS"). |
| osv | string | Optional | - | Device operating system version (e.g., "3.1.2"). |
| hwv | string | Optional | - | Hardware version of the device (e.g., "5S" for iPhone 5S). |
| h | integer | Optional | - | Physical height of the screen in pixels. |
| w | integer | Optional | - | Physical width of the screen in pixels. |
| ppi | integer | Optional | - | Screen size as pixels per linear inch. |
| pxratio | float | Optional | - | The ratio of physical pixels to device independent pixels. |
| js | integer | Optional | - | Support for JavaScript, where 0 = no, 1 = yes. |
| geofetch | integer | Optional | - | Indicates if the geolocation API will be available to JavaScript code running in the banner, where 0 = no, 1 = yes. |
| flashver | string | Optional | - | Version of Flash supported by the browser. |
| language | string | Optional | - | Browser language using ISO-639-1-alpha-2. |
| carrier | string | Optional | - | Carrier or ISP (e.g., "VERIZON") using exchange curated string names which should be published to bidders a priori. |
| mccmnc | string | Optional | - | Mobile carrier as the concatenated MCC-MNC code (e.g., "310-005" identifies Verizon Wireless CDMA in the USA). Refer to https://en.wikipedia.org/wiki/Mobile_country_code for further examples. Note that the dash between the MCC and MNC parts is required to remove parsing ambiguity. |
| connectiontype | integer | Optional | - | Network connection type. Refer to List 5.22. |
| ifa | string | Optional | - | ID sanctioned for advertiser use in the clear (i.e., not hashed). |
| didsha1 | string | Optional | - | Hardware device ID (e.g., IMEI); hashed via SHA1. |
| didmd5 | string | Optional | - | Hardware device ID (e.g., IMEI); hashed via MD5. |
| dpidsha1 | string | Optional | - | Platform device ID (e.g., Android ID); hashed via SHA1. |
| dpidmd5 | string | Optional | - | Platform device ID (e.g., Android ID); hashed via MD5. |
| macsha1 | string | Optional | - | MAC address of the device; hashed via SHA1. |
| macmd5 | string | Optional | - | MAC address of the device; hashed via MD5. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 19 Object: Geo

This object encapsulates various methods for specifying a geographic location. When subordinate to a Device object, it indicates the location of the device which can also be interpreted as the user's current location. When subordinate to a User object, it indicates the location of the user's home base (i.e., not necessarily their current location).

The lat/lon attributes should only be passed if they conform to the accuracy depicted in the type attribute. For example, the centroid of a geographic region such as postal code should not be passed.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| lat | float | Optional | - | Latitude from -90.0 to +90.0, where negative is south. |
| lon | float | Optional | - | Longitude from -180.0 to +180.0, where negative is west. |
| type | integer | Optional | - | Source of location data; recommended when passing lat/lon. Refer to List 5.20. |
| accuracy | integer | Optional | - | Estimated location accuracy in meters; recommended when lat/lon are specified and derived from a device's location services (i.e., type = 1). Note that this is the accuracy as reported from the device. Consult OS specific documentation (e.g., Android, iOS) for exact interpretation. |
| lastfix | integer | Optional | - | Number of seconds since this geolocation fix was established. Note that devices may cache location data across multiple fetches. Ideally, this value should be from the time the actual fix was taken. |
| ipservice | integer | Optional | - | Service or provider used to determine geolocation from IP address if applicable (i.e., type = 2). Refer to List 5.23. |
| country | string | Optional | - | Country code using ISO-3166-1-alpha-3. |
| region | string | Optional | - | Region code using ISO-3166-2; 2-letter state code if USA. |
| regionfips104 | string | Optional | - | Region of a country using FIPS 10-4 notation. While OpenRTB supports this attribute, it has been withdrawn by NIST in 2008. |
| metro | string | Optional | - | Google metro code; similar to but not exactly Nielsen DMAs. See Appendix A for a link to the codes. |
| city | string | Optional | - | City using United Nations Code for Trade & Transport Locations. See Appendix A for a link to the codes. |
| zip | string | Optional | - | Zip or postal code. |
| utcoffset | integer | Optional | - | Local time as the number +/- of minutes from UTC. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 20 Object: User

This object contains information known or derived about the human user of the device (i.e., the audience for advertising). The user id is an exchange artifact and may be subject to rotation or other privacy policies. However, this user ID must be stable long enough to serve reasonably as the basis for frequency capping and retargeting.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Recommended | - | Exchange-specific ID for the user. At least one of id or buyeruid is recommended. |
| buyeruid | string | Recommended | - | Buyer-specific ID for the user as mapped by the exchange for the buyer. At least one of buyeruid or id is recommended. |
| yob | integer | Optional | - | Year of birth as a 4-digit integer. |
| gender | string | Optional | - | Gender, where "M" = male, "F" = female, "O" = known to be other (i.e., omitted is unknown). |
| keywords | string | Optional | - | Comma separated list of keywords, interests, or intent. |
| customdata | string | Optional | - | Optional feature to pass bidder data that was set in the exchange's cookie. The string must be in base85 cookie safe characters and be in any format. Proper JSON encoding must be used to include "escaped" quotation marks. |
| geo | object | Optional | - | Location of the user's home base defined by a Geo object (Section 19). This is not necessarily their current location. |
| data | object array | Optional | - | Additional user data. Each Data object (Section 21) represents a different data source. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 21 Object: Data

The data and segment objects together allow additional data about the related object (e.g., user, content) to be specified. This data may be from multiple sources whether from the exchange itself or third parties as specified by the id field. A bid request can mix data objects from multiple providers. The specific data providers in use should be published by the exchange a priori to its bidders.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Optional | - | Exchange-specific ID for the data provider. |
| name | string | Optional | - | Exchange-specific name for the data provider. |
| segment | object array | Optional | - | Array of Segment (Section 22) objects that contain the actual data values. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## 22 Object: Segment

Segment objects are essentially key-value pairs that convey specific units of data. The parent Data object is a collection of such values from a given data provider. The specific segment names and value options must be published by the exchange a priori to its bidders.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Optional | - | ID of the data segment specific to the data provider. |
| name | string | Optional | - | Name of the data segment specific to the data provider. |
| value | string | Optional | - | String representation of the data segment value. |
| ext | object | Optional | - | Placeholder for exchange-specific extensions to OpenRTB. |

## Sample

### Banner
```json
{
  "id": "1-1790905403968-3098-605-3193763921",
  "at": 2,
  "imp": [
    {
      "id": "320x50",
      "instl": 0,
      "bidfloor": 0.25,
      "bidfloorcur": "USD",
      "tagid": "7883",
      "secure": 1,
      "displaymanager": "admize",
      "displaymanagerver": "1.0.0",
      "banner": {
        "id": "banner",
        "w": 320,
        "h": 50
      }
    }
  ],
  "tmax": 220,
  "cur": ["USD"],
  "badv": ["app.bav-test.com"],
  "bapp": [],
  "bcat": ["IAB24", "IAB7-5", "IAB3-7", "IAB25-3", "IAB26-2"],
  "app": {
    "id": "0001",
    "name": "com.test.app",
    "bundle": "com.test.app",
    "storeurl": "https://play.google.com/store/apps/details?id=com.hellomarket.com.test.app",
    "publisher": {
      "id": "545dc211-f880-4385-a67e-ea58e150992f",
      "name": "admize",
      "ext": { "sid": 6 }
    }
  },
  "device": {
    "ua": "Dalvik/2.1.0 (Linux; U; Android 16; SM-S947N Build/BP4A.251205.006)",
    "ip": "211.234.197.46",
    "devicetype": 1,
    "make": "samsung",
    "model": "SM-S947N",
    "os": "Android",
    "osv": "16",
    "language": "ko",
    "ifa": "e02e51d9-975d-4976-9e29-775a672dd371",
    "connectiontype": 1,
    "geo": {
      "country": "KOR",
      "city": "Gwangjin-gu"
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
            "sid": "12405",
            "rid": "1790905403253383770",
            "hp": 1
          },
          {
            "asi": "admize.io",
            "sid": "p_1875135558794156",
            "rid": "1790905403253383770",
            "hp": 1
          }
        ]
      }
    }
  }
}
```

### Video
```json
{
  "id": "1-1790906127770-387656-9270-32559831270",
  "at": 1,
  "imp": [
    {
      "id": "e5b0372c-a1ba-42c1-9fe8-5b5a113254d1",
      "instl": 0,
      "bidfloor": 1.1213,
      "bidfloorcur": "USD",
      "tagid": "4760895",
      "secure": 1,
      "displaymanager": "admize",
      "displaymanagerver": "1.0.0",
      "video": {
        "mimes": ["video/mp4"],
        "minduration": 5,
        "maxduration": 30,
        "protocols": [2],
        "w": 240,
        "h": 320,
        "startdelay": 0,
        "linearity": 1,
        "playbackmethod": [2, 4],
        "delivery": [2],
        "companiontype": [1, 2, 3],
        "placement": 5,
        "playbackend": 1
      }
    }
  ],
  "tmax": 220,
  "cur": ["USD"],
  "badv": ["app.bav-test.com"],
  "bapp": [],
  "bcat": ["IAB26-2", "IAB26-1", "IAB9-9", "IAB25-4",],
  "app": {
    "id": "0001",
    "name": "com.test.app",
    "bundle": "com.test.app",
    "storeurl": "https://play.google.com/store/apps/details?id=com.hellomarket.com.test.app",
    "publisher": {
      "id": "892e9303-2004-4945-9d26-80eb8b9639cb",
      "name": "admize",
      "ext": { "sid": 26 }
    }
  },
  "device": {
    "ua": "Dalvik/2.1.0 (Linux; U; Android 14; SM-M236L Build/UP1A.231005.007)",
    "ip": "58.29.178.131",
    "devicetype": 1,
    "make": "samsung",
    "model": "SM-M236L",
    "os": "Android",
    "osv": "14",
    "language": "ko",
    "ifa": "d598192f-4c84-4d7b-3e40-2a90ii4f480d",
    "connectiontype": 1,
    "geo": {
      "country": "KOR",
      "city": "-"
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
            "sid": "11197",
            "rid": "1790906127059967655",
            "hp": 1
          },
          {
            "asi": "admize.io",
            "sid": "p_1875135558794156",
            "rid": "1790906127059967655",
            "hp": 1
          }
        ]
      }
    }
  }
}
```