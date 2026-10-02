# Object Specifications

The subsections that follow define each of the objects in the bid response model. Several conventions are used throughout:

- Attributes are "required" if their omission would technically break the protocol.
- Some optional attributes are denoted "recommended" due to their elevated business importance.
- Unless a default value is explicitly specified, an omitted attribute is interpreted as "unknown".

## 1 Object: BidResponse

This object is the top-level bid response object (i.e., the unnamed outer JSON object). The `id` attribute is a reflection of the bid request ID for logging purposes. Similarly, `bidid` is an optional response tracking ID for bidders. If specified, it can be included in the subsequent win notice call if the bidder wins. At least one seatbid object is required, which contains at least one bid for an impression. Other attributes are optional.

To express a "no-bid", the options are to return an empty response with HTTP 204. Alternately if the bidder wishes to convey to the exchange a reason for not bidding, just a BidResponse object is returned with a reason code in the nbr attribute.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Required | - | ID of the bid request to which this is a response. |
| seatbid | object array | Required | - | Array of seatbid objects; 1+ required if a bid is to be made. |
| bidid | string | Optional | - | Bidder generated response ID to assist with logging/tracking. |
| cur | string | Optional | "USD" | Bid currency using ISO-4217 alpha codes. |
| customdata | string | Optional | - | Optional feature to allow a bidder to set data in the exchange's cookie. The string must be in base85 cookie safe characters and be in any format. Proper JSON encoding must be used to include "escaped" quotation marks. |
| nbr | integer | Optional | - | Reason for not bidding. Refer to List 5.24. |
| ext | object | Optional | - | Placeholder for bidder-specific extensions to OpenRTB. |

## 2 Object: SeatBid

A bid response can contain multiple SeatBid objects, each on behalf of a different bidder seat and each containing one or more individual bids. If multiple impressions are presented in the request, the `group` attribute can be used to specify if a seat is willing to accept any impressions that it can win (default) or if it is only interested in winning any if it can win them all as a group.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| bid | object array | Required | - | Array of 1+ Bid objects (Section 3) each related to an impression. Multiple bids can relate to the same impression. |
| seat | string | Optional | - | ID of the buyer seat (e.g., advertiser, agency) on whose behalf this bid is made. |
| group | integer | Optional | 0 | 0 = impressions can be won individually; 1 = impressions must be won or lost as a group. |
| ext | object | Optional | - | Placeholder for bidder-specific extensions to OpenRTB. |

## 3 Object: Bid

A SeatBid object contains one or more Bid objects, each of which relates to a specific impression in the bid request via the `impid` attribute and constitutes an offer to buy that impression for a given price.

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| id | string | Required | - | Bidder generated bid ID to assist with logging/tracking. |
| impid | string | Required | - | ID of the Imp object in the related bid request. |
| price | float | Required | - | Bid price expressed as CPM although the actual transaction is for a unit impression only. Note that while the type indicates float, integer math is highly recommended when handling currencies (e.g., BigDecimal in Java). |
| nurl | string | Optional | - | Win notice URL called by the exchange if the bid wins (not necessarily indicative of a delivered, viewed, or billable ad); optional means of serving ad markup. Substitution macros (Section 4.4) may be included in both the URL and optionally returned markup. |
| burl | string | Optional | - | Billing notice URL called by the exchange when a winning bid becomes billable based on exchange-specific business policy (e.g., typically delivered, viewed, etc.). Substitution macros (Section 4.4) may be included. |
| lurl | string | Optional | - | Loss notice URL called by the exchange when a bid is known to have been lost. Substitution macros (Section 4.4) may be included. Exchange-specific policy may preclude support for loss notices or the disclosure of winning clearing prices resulting in ${AUCTION_PRICE} macros being removed (i.e., replaced with a zero-length string). |
| adm | string | Optional | - | Optional means of conveying ad markup in case the bid wins; supersedes the win notice if markup is included in both. Substitution macros (Section 4.4) may be included. |
| adid | string | Optional | - | ID of a preloaded ad to be served if the bid wins. |
| adomain | string array | Optional | - | Advertiser domain for block list checking (e.g., "ford.com"). This can be an array of for the case of rotating creatives. Exchanges can mandate that only one domain is allowed. |
| bundle | string | Optional | - | A platform-specific application identifier intended to be unique to the app and independent of the exchange. On Android, this should be a bundle or package name (e.g., com.foo.mygame). On iOS, it is a numeric ID. |
| iurl | string | Optional | - | URL without cache-busting to an image that is representative of the content of the campaign for ad quality/safety checking. |
| cid | string | Optional | - | Campaign ID to assist with ad quality checking; the collection of creatives for which iurl should be representative. |
| crid | string | Optional | - | Creative ID to assist with ad quality checking. |
| tactic | string | Optional | - | Tactic ID to enable buyers to label bids for reporting to the exchange the tactic through which their bid was submitted. The specific usage and meaning of the tactic ID should be communicated between buyer and exchanges a priori. |
| cat | string array | Optional | - | IAB content categories of the creative. Refer to List 5.1. |
| attr | integer array | Optional | - | Set of attributes describing the creative. Refer to List 5.3. |
| api | integer | Optional | - | API required by the markup if applicable. Refer to List 5.6. |
| protocol | integer | Optional | - | Video response protocol of the markup if applicable. Refer to List 5.8. |
| qagmediarating | integer | Optional | - | Creative media rating per IQG guidelines. Refer to List 5.19. |
| language | string | Optional | - | Language of the creative using ISO-639-1-alpha-2. The non-standard code "xx" may also be used if the creative has no linguistic content (e.g., a banner with just a company logo). |
| dealid | string | Optional | - | Reference to the deal.id from the bid request if this bid pertains to a private marketplace direct deal. |
| w | integer | Optional | - | Width of the creative in device independent pixels (DIPS). |
| h | integer | Optional | - | Height of the creative in device independent pixels (DIPS). |
| wratio | integer | Optional | - | Relative width of the creative when expressing size as a ratio. Required for Flex Ads. |
| hratio | integer | Optional | - | Relative height of the creative when expressing size as a ratio. Required for Flex Ads. |
| exp | integer | Optional | - | Advisory as to the number of seconds the bidder is willing to wait between the auction and the actual impression. |
| ext | object | Optional | - | Placeholder for bidder-specific extensions to OpenRTB. |

## 3-1 Object: Bid.Ext

| Attribute | Type | Required/Optional | Default | Description |
|-----------|------|------------------|---------|-------------|
| clicktrackers | string array | Optional | - | ClickTracking Url. |

## Substitution Macros

Macros that can be used in URLs for tracking and reporting.

| Macro | Description |
|-------|-------------|
| ${AUCTION_ID} | ID of the bid request; from BidRequest.id attribute |
| ${AUCTION_SEAT_ID} | ID of the bidder seat for whom the bid was made |
| ${AUCTION_AD_ID} | ID of the ad markup the bidder wishes to serve; from bid.adid attribute |
| ${AUCTION_BID_ID} | ID of the bid; from BidResponse.bidid attribute |
| ${AUCTION_IMP_ID} | ID of the impression just won; from imp.id attribute |
| ${AUCTION_CURRENCY} | The currency used in the bid (explicit or implied); for confirmation only |
| ${AUCTION_PRICE} | Clearing price using the same currency and units as the bid |
| ${AUCTION_PRICE:B64} | Base64 encoded clearing price |

## Sample

### Banner
```json
{
  "id": "4b64365402e54e0869e1d39c2ee4c432",
  "seatbid": [
    {
      "bid": [
        {
          "id": "1-1644894781441-161-23-2446841",
          "impid": "06a5f7ca-0f13-4ae1-6a6e-15ffef02c650",
          "price": 0.3068,
          "adid": "495E8194:1644894781:0363905980",
          "adm": "<!doctype html>\n<html>\n<head>\n  <meta charset=\"utf-8\">\n  <meta name=\"viewport\" content=\"width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no\">\n  <link rel=\"icon\" href=\"data:,\">\n  <style>\n    BODY {\n      margin: 0;\n      padding: 0;\n    }\n    .container {\n      width: 100%;\n      height: 100%;\n    }\n    .block {\n      width: 320px;\n      height: 50px;\n      margin: 0 auto;\n      overflow: hidden;\n    }\n  </style>\n</head>\n<body>\n<div class=\"container\">\n  <div class=\"block\">\n    <iframe id=\"Admize_area\" name=\"Admize_area\" style=\"width:320px; height:50px; margin:0px; padding:0px; border: none;\"></iframe>\n  </div>\n</div>\n<script>\n  let decode = function(input) {\n    input = input.replace(/-/g, '+').replace(/_/g, '/');\n    let pad = input.length % 4;\n    if (pad) {\n      if (pad === 1) {\n        throw new Error('InvalidLengthError: Input base64url string is the wrong length to determine padding');\n      }\n      input += new Array(5 - pad).join('=');\n    }\n    return input;\n  };\n  let adm = 'CjxodG1sPjxoZWFkPjxtZXRhIGh0dHAtZXF1aXY9IkNvbnRlbnQtVHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04Ij4KCiAgICAKICAgIDxtZXRhIGlkPSJ2cCIgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IiI-Cgo8bGluayB0eXBlPSJ0ZXh0L2NzcyIgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL2ltYWdlLmNhdWx5LmNvLmtyL3JpY2hhZC90ZXN0L2NoYW5nanUvb3B0b3V0L2Nfb3B0b3V0LmNzcyIgPgoKPHNjcmlwdCBzcmM9Imh0dHBzOi8vYWpheC5nb29nbGVhcGlzLmNvbS9hamF4L2xpYnMvanF1ZXJ5LzEuMTEuMi9qcXVlcnkubWluLmpzIj48L3NjcmlwdD4KPHNjcmlwdCBpZD0iY19vcHRvdXQiIHNyYz0iaHR0cDovL2ltYWdlLmNhdWx5LmNvLmtyL29wdG91dC9jX29wdG91dC5qcz9hZGlkPTYzODM0M2E2LWU5MTAtNGYzMy1iODM2LTAwMGRlNzRiNGMxOCZwbGF0Zm9ybWNkPTImYWRjZD00NTQ2NTEiPjwvc2NyaXB0PiAKPGRpdiBzdHlsZT0icG9zaXRpb246IHJlbGF0aXZlOyI-Cgk8YSBocmVmPSJodHRwOi8vdWF0LWNsaWNrLmZzbnN5cy5jb20vQ2F1bHlDbGljaz9zZGtfdHlwZT1uYXRpdmUmY29kZT1UUW53N0NqOSZpZD00NTQ2NTYmdmVyc2lvbj01LjEuMSZzZGtfdmVyc2lvbj00LjUuMSZwbGF0Zm9ybT1BbmRyb2lkJnNjb2RlPTYzODM0M2E2LWU5MTAtNGYzMy1iODM2LTAwMGRlNzRiNGMxOCZpc2VyaWFsPTQ5NUU4MTk0OjE2NDQ4OTQ3ODE6MDM2MzgwNTk3MCZhZF9mb3JtPWJhbm5lciZuZXR3b3JrPVdJRkkmcGF5X3R5cGU9Y3BjJnRhcmdldF91cmw9aHR0cCUzQSUyRiUyRnVhdC1jbGljay5mc25zeXMuY29tJTJGQ2F1bHlDbGljayUzRnNka190eXBlJTNEbmF0aXZlJTI2Y29kZSUzRFRRbnc3Q2o5JTI2aWQlM0Q0NTQ2NTYlMjZ2ZXJzaW9uJTNENS4xLjElMjZzZGtfdmVyc2lvbiUzRDQuNS4xJTI2cGxhdGZvcm0lM0RBbmRyb2lkJTI2c2NvZGUlM0Q2MzgzNDNhNi1lOTEwLTRmMzMtYjgzNi0wMDBkZTc0YjRjMTglMjZpc2VyaWFsJTNENDk1RTgxOTQlM0ExNjQ0ODk0NzgxJTNBMDM2MzgwNTk3MCUyNmFkX2Zvcm0lM0RiYW5uZXIlMjZuZXR3b3JrJTNEV0lGSSUyNnBheV90eXBlJTNEY3BjJTI2dGFyZ2V0X3VybCUzRGh0dHBzJTI1M0ElMjUyRiUyNTJGd3d3LmV4YW1wbGUuY29tJTI1MkYlMjZjbGlja19hY3Rpb25fcGFyYW0xJTNEaHR0cHMlMjUzQSUyNTJGJTI1MkZ3d3cuZXhhbXBsZS5jb20lMjUyRiUyNmNfY29udHJvbCUzRGwmY2xpY2tfYWN0aW9uX3BhcmFtMT1odHRwJTNBJTJGJTJGdWF0LWNsaWNrLmZzbnN5cy5jb20lMkZDYXVseUNsaWNrJTNGc2RrX3R5cGUlM0RuYXRpdmUlMjZjb2RlJTNEVFFudzdDajklMjZpZCUzRDQ1NDY1NiUyNnZlcnNpb24lM0Q1LjEuMSUyNnNka192ZXJzaW9uJTNENC41LjElMjZwbGF0Zm9ybSUzREFuZHJvaWQlMjZzY29kZSUzRDYzODM0M2E2LWU5MTAtNGYzMy1iODM2LTAwMGRlNzRiNGMxOCUyNmlzZXJpYWwlM0Q0OTVFODE5NCUzQTE2NDQ4OTQ3ODElM0EwMzYzODA1OTcwJTI2YWRfZm9ybSUzRGJhbm5lciUyNm5ldHdvcmslM0RXSUZJJTI2cGF5X3R5cGUlM0RjcGMlMjZ0YXJnZXRfdXJsJTNEaHR0cHMlMjUzQSUyNTJGJTI1MkZ3d3cuZXhhbXBsZS5jb20lMjUyRiUyNmNsaWNrX2FjdGlvbl9wYXJhbTElM0RodHRwcyUyNTNBJTI1MkYlMjUyRnd3dy5leGFtcGxlLmNvbSUyNTJGJTI2Y19jb250cm9sJTNEbCIgdGFyZ2V0PSJfYmxhbmsiPgoJCTxpbWcgc3R5bGU9ImJvcmRlcjogMDsiIGJvcmRlcj0iMCIgd2lkdGg9IjMyMCIgaGVpZ2h0PSI1MCIgc3JjPSJodHRwOi8vY2F1bHkxNDIuZnNuc3lzLmNvbS9pY29uLzIwMjEvMDQvNTRhZGMxZmIxZGRhNGM2M2FjNzA4OGViMDQ5ZjA3YzJfMTY3MDQuanBnIiBhbHQ9IiI-PC9pbWc-Cgk8L2E-Cgk8aW1nIHN0eWxlPSJib3JkZXI6IDA7IiBib3JkZXI9IjAiIHdpZHRoPSIwIiBoZWlnaHQ9IjAiIHNyYz0iaHR0cDovL2NhdWx5MTQxLmZzbnN5cy5jb206MTI0NDgvY2F1bHlEc3BJbmZvcm0_c2RrX3R5cGU9bmF0aXZlJmFkc19jZD00NTQ2NTYmdmVyc2lvbj01LjEuMSZzZGtfdmVyc2lvbj00LjUuMSZwbGF0Zm9ybT1BbmRyb2lkJmNvZGU9VFFudzdDajkmbW9kZWw9YW5kcm9pZCZzY29kZT02MzgzNDNhNi1lOTEwLTRmMzMtYjgzNi0wMDBkZTc0YjRjMTgmc2NvZGVfdHlwZT1naWQmYWRfc2hhcGU9YmFubmVyJmFkX2Zvcm09YmFubmVyJmlzZXJpYWw9NDk1RTgxOTQlM0ExNjQ0ODk0NzgxJTNBMDM2MzgwNTk3MCZ2aXNpYmxlPVkmcGFrZXk9JmJpZF9mbG9vcj0wLjEzJnByaWNlPTAuMzQ0NSZ1c2VfYnVybD1OJmF1Y3Rpb25fcHJpY2U9MC40MzgyJmF1Y3Rpb25pZD0xLTE2NDQ4OTQ3ODE0NDEtMTYxLTIzLTI0NDY4NDEiIGFsdD0iIi8-Cgk8c3BhbiBzdHlsZT0icG9zaXRpb246YWJzb2x1dGU7IGxlZnQ6MzA1cHg7Ij4KCQk8ZGl2IGNsYXNzPSJvcHQtb3V0Ij4gCgkJCTxkaXYgY2xhc3M9ImNpcmNsZSIgPgoJCQk8L2Rpdj4gCgkJCTxkaXYgY2xhc3M9ImJhciIgc3R5bGU9ImRpc3BsYXk6IGJsb2NrOyB3aWR0aDogNTRweDsgZGlzcGxheTogbm9uZTsiPgoJCQkJPGltZyBzcmM9Imh0dHBzOi8vaW1hZ2UuY2F1bHkuY28ua3Ivb3B0b3V0L2NhdWx5X2Iuc3ZnIiBhbHQ9IiI-CgkJCTwvZGl2PiAKCQkJPGRpdiBjbGFzcz0iZG90IiA-CgkJCTwvZGl2PgoJCQk8ZGl2IGNsYXNzPSJsaW5lIiA-CgkJCTwvZGl2PgoJCTwvZGl2PgoJPC9zcGFuPgo8L2Rpdj4KPC9odG1sPgo';\n  let doc = document.getElementById('Admize_area').contentWindow.document;\n  doc.open();\n  doc.write('<meta charset=\"utf-8\"><meta name=\"viewport\" content=\"width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no\">' + decodeURIComponent(escape(window.atob(decode(adm)))));\n  let style = document.createElement('style');\n  style.textContent = 'body {margin:0;padding:0}';\n  doc.head.appendChild(style);\n</script>\n<script>\n  function changeParameter(sourceUrl, targetParam, replace) {\n    let index = sourceUrl.indexOf(targetParam);\n    let changedUrl = \"\";\n    if (index >= 0) {\n      let lastIndex = sourceUrl.indexOf(\"&\", index + targetParam.length);\n      if (lastIndex >= 0) {\n        changedUrl = sourceUrl.substring(0, index) + replace + sourceUrl.substring(lastIndex);\n      } else {\n        changedUrl = sourceUrl.substring(0, index) + replace;\n      }\n    } else {\n      changedUrl = sourceUrl + \"&\" + replace;\n    }\n    return changedUrl;\n  }\n\n  function sendBeacon(url, param1, param2) {\n    let beaconUrl = changeParameter(url, param1, param2);\n    let beacon = document.createElement(\"img\");\n    beacon.style[\"display\"] = \"none\";\n    beacon.src = beaconUrl;\n    document.getElementsByTagName(\"BODY\")[0].appendChild(beacon);\n  }\n\n  function createBeacon(impUrl) {\n    sendBeacon(impUrl, \"inform_type=\", \"inform_type=\");\n    let intervalFlag = false;\n    if (intervalFlag == false) {\n      let intervalValue = setInterval(function() {\n        if (window.innerWidth > 0) {\n          sendBeacon(impUrl, \"inform_type=\", \"inform_type=cn\");\n          setTimeout(function() {\n            sendBeacon(impUrl, \"inform_type=\", \"inform_type=cy\");\n          }, 1000);\n          clearInterval(intervalValue);\n        }\n        intervalFlag = true;\n      }, 300);\n    }\n  }\n\n  createBeacon('https://test-event.admize.io/imp/ssp/v1/1-1644894781441-161-23-2446841?ap=${AUCTION_PRICE}');\n</script>\n</body>\n</html>\n",
          "adomain": [
            "example.com"
          ],
          "cid": "23501",
          "crid": "454651",
          "cat": [
            "IAB1-1"
          ],
          "w": 320,
          "h": 50,
          "ext": {
            "clicktrackers": [
              "https://test-click.Admize.io/v1/click/1-1663565140182-54-46-2076059"
            ]
          }
        }
      ]
    }
  ]
}
```

### Video
```json
{
  "id": "7c1f2a9e3b8d4e6fa0b5c2d9e8f71a34",
  "bidid": "b-7c1f2a9e3b8d-0001",
  "cur": "USD",
  "seatbid": [
    {
      "seat": "seat-1",
      "bid": [
        {
          "id": "bid-2f6b9c1e-0001",
          "impid": "2f6b9c1e-4a7d-4c3e-9b8a-5d1e7f2a6c90",
          "price": 0.5,
          "nurl": "https://win.dsp.example.com/win?bid=bid-2f6b9c1e-0001&price=${AUCTION_PRICE}",
          "adm": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<VAST version=\"2.0\">\n  <Ad id=\"ad-5001\">\n    <InLine>\n      <AdSystem version=\"1.0\">Test DSP</AdSystem>\n      <AdTitle>Test Video Ad</AdTitle>\n      <Error><![CDATA[https://track.dsp.example.com/error?bid=bid-2f6b9c1e-0001&code=[ERRORCODE]]]></Error>\n      <Impression><![CDATA[https://track.dsp.example.com/imp?bid=bid-2f6b9c1e-0001&price=${AUCTION_PRICE}]]></Impression>\n      <Impression id=\"admize-impression\"><![CDATA[https://test-event.admize.io/imp/ssp/v1/bid-2f6b9c1e-0001?ap=${AUCTION_PRICE}]]></Impression>\n      <Creatives>\n        <Creative id=\"cr-5001\" sequence=\"1\">\n          <Linear>\n            <Duration>00:00:15</Duration>\n            <TrackingEvents>\n              <Tracking event=\"start\"><![CDATA[https://track.dsp.example.com/event?bid=bid-2f6b9c1e-0001&e=start]]></Tracking>\n              <Tracking event=\"firstQuartile\"><![CDATA[https://track.dsp.example.com/event?bid=bid-2f6b9c1e-0001&e=firstQuartile]]></Tracking>\n              <Tracking event=\"midpoint\"><![CDATA[https://track.dsp.example.com/event?bid=bid-2f6b9c1e-0001&e=midpoint]]></Tracking>\n              <Tracking event=\"thirdQuartile\"><![CDATA[https://track.dsp.example.com/event?bid=bid-2f6b9c1e-0001&e=thirdQuartile]]></Tracking>\n              <Tracking event=\"complete\"><![CDATA[https://track.dsp.example.com/event?bid=bid-2f6b9c1e-0001&e=complete]]></Tracking>\n              <Tracking event=\"close\"><![CDATA[https://track.dsp.example.com/event?bid=bid-2f6b9c1e-0001&e=close]]></Tracking>\n            </TrackingEvents>\n            <VideoClicks>\n              <ClickThrough><![CDATA[https://www.example.com/landing]]></ClickThrough>\n              <ClickTracking><![CDATA[https://track.dsp.example.com/click?bid=bid-2f6b9c1e-0001]]></ClickTracking>\n            </VideoClicks>\n            <MediaFiles>\n              <MediaFile delivery=\"progressive\" type=\"video/mp4\" width=\"320\" height=\"480\" bitrate=\"800\" scalable=\"true\" maintainAspectRatio=\"true\"><![CDATA[https://cdn.dsp.example.com/video/320x480_15s.mp4]]></MediaFile>\n            </MediaFiles>\n          </Linear>\n        </Creative>\n        <Creative id=\"cr-5001-companion\" sequence=\"1\">\n          <CompanionAds>\n            <Companion width=\"320\" height=\"480\">\n              <StaticResource creativeType=\"image/jpeg\"><![CDATA[https://cdn.dsp.example.com/companion/320x480.jpg]]></StaticResource>\n              <CompanionClickThrough><![CDATA[https://www.example.com/landing]]></CompanionClickThrough>\n            </Companion>\n          </CompanionAds>\n        </Creative>\n      </Creatives>\n    </InLine>\n  </Ad>\n</VAST>",
          "adid": "ad-5001",
          "adomain": ["example.com"],
          "cid": "cmp-5001",
          "crid": "cr-5001",
          "cat": ["IAB9"],
          "protocol": 2,
          "w": 320,
          "h": 480
        }
      ]
    }
  ]
}
```