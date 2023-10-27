Next.js 에서 Vworld를 이용해 지도 구현하기 - Openlayers 라이브러리를 이용한 후, 타일로 Vworld의 지도 이미지를 씌우는 방법 

0. 사용 전에 Vworld에서 API 키를 사용목적에 따라 발급받기. 

1.  npm i ol     // openlayers 라이브러리 설치하기
2.  layout.tsx 에 MapProvider 추가
3. MAP Context 만들기
```tsx
"use client";

import React from "react";

  

const MapContext = React.createContext({});

export default MapContext;
```
5. MapProvider에 넣을 MAP 컴포넌트 만들기 
```tsx
"use client";

import React, { useState, useEffect } from "react";
import MapContext from "./mapContext";
import "ol/ol.css";
import { Map, View } from "ol";
import { defaults } from "ol/control";
import { fromLonLat, get } from "ol/proj";
import { Tile, Vector } from "ol/layer";
import { OSM, XYZ } from "ol/source";
import { FaSchool } from "react-icons/fa6";
import Point from "ol/geom/Point.js";
import Feature from "ol/Feature.js";
import { Style, Text, Icon, Fill } from "ol/style";
import VectorSource from "ol/source/Vector.js";

const MapComponent = ({ children }) => {

  const [mapObj, setMapObj] = useState({});

  

  useEffect(() => {

    const map = new Map({

      controls: defaults({ zoom: true, rotate: true }).extend([]),

      layers: [

        new Tile({

          source: new OSM(),

        }),

        new Tile({

          name: "Base",

          visible: true,

          source: new XYZ({

            url: "http://api.vworld.kr/req/wmts/1.0.0/{API 키}/Base/{z}/{y}/{x}.png",

          }),

        }),

      ],

      target: "map",

      view: new View({

        projection: get("EPSG:3857"),

        center: fromLonLat([128.681882, 35.2279728], get("EPSG:3857")),

        zoom: 14,

      }),

    });


    setMapObj({ map });

    return () => map.setTarget(undefined);

  }, []);


  return (

    <MapContext.Provider value={{ mapObj }}>{children}</MapContext.Provider>

  );

};

  

export default MapComponent;
```

4. MAP을 사용할 페이지의 클라이언트 컴포넌트에서 필요한 레이어 등을 만들고 넣기 
```tsx

```