---
layout: post
title: Pretty Print JSON Object
date: 2018-09-09 14:00:00 +0900
categories: blog
tags: javascript
---

API 사용 중 JSON으로 된 결과를 받아오는 경우 단순히 console.log(json)를 찍으면 아래와 같이 나온다.

```javascript
{"results":[{"address_components":[{"long_name":"Seongnam-si","short_name":"Seongnam-si","types":["locality","political"]},{"long_name":"Gyeonggi-do","short_name":"Gyeonggi-do","types":["administrative_area_level_1","political"]},{"long_name":"South Korea","short_name":"KR","types":["country","political"]}],"formatted_address":"Seongnam-si, Gyeonggi-do, South Korea","geometry":{"bounds":{"northeast":{"lat":37.4756297,"lng":127.1974457},"southwest":{"lat":37.3330741,"lng":127.0235872}},"location":{"lat":37.4449168,"lng":127.1388684},"location_type":"APPROXIMATE","viewport":{"northeast":{"lat":37.4654,"lng":127.1796},"southwest":{"lat":37.3351,"lng":127.1023}}},"place_id":"ChIJvyAScPGnfDUR1JfjvGZWMVA","types":["locality","political"]}],"status":"OK"}
```



위와 같이 나오면 indentation도 하나도 되어있지 않아서 보기가 힘들고 디버깅 또한 너무 힘들다.이런 경우 외부 라이브러리를 쓰는 것도 한 방법이고 아니면 단순하게 아래와 같이 JSON.stringify()를 쓰는 것도 좋은 방법이다.

```javascript
console.log(JSON.stringify(json, undefined, 2));
```



세번째 인자는 indentation을 가리킨다. 위와 같이 실행하면 결과는 다음과 같다.

```javascript
{
  "results": [
    {
      "address_components": [
        {
          "long_name": "Seongnam-si",
          "short_name": "Seongnam-si",
          "types": [
            "locality",
            "political"
          ]
        },
        {
          "long_name": "Gyeonggi-do",
          "short_name": "Gyeonggi-do",
          "types": [
            "administrative_area_level_1",
            "political"
          ]
        },
        {
          "long_name": "South Korea",
          "short_name": "KR",
          "types": [
            "country",
            "political"
          ]
        }
      ],
      "formatted_address": "Seongnam-si, Gyeonggi-do, South Korea",
      "geometry": {
        "bounds": {
          "northeast": {
            "lat": 37.4756297,
            "lng": 127.1974457
          },
          "southwest": {
            "lat": 37.3330741,
            "lng": 127.0235872
          }
        },
        "location": {
          "lat": 37.4449168,
          "lng": 127.1388684
        },
        "location_type": "APPROXIMATE",
        "viewport": {
          "northeast": {
            "lat": 37.4654,
            "lng": 127.1796
          },
          "southwest": {
            "lat": 37.3351,
            "lng": 127.1023
          }
        }
      },
      "place_id": "ChIJvyAScPGnfDUR1JfjvGZWMVA",
      "types": [
        "locality",
        "political"
      ]
    }
  ],
  "status": "OK"
}
```



MDN에 따르면 JSON.stringify()는 다음과 같은 인자를 받는다.

```javascript
JSON.stringify(value[, replacer[, space]]);
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify
```