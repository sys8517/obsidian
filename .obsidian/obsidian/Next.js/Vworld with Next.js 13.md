Next.js 13 에서 Vworld를 이용해 지도 구현하기 - Openlayers 라이브러리를 이용한 후, 타일로 Vworld의 지도 이미지를 씌우는 방법 

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

4. MapProvider에 넣을 MAP Component 만들기 
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

5. MAP을 사용할 페이지의 클라이언트 컴포넌트에서 필요한 레이어 등을 만들고 넣기 
```tsx
"use client";

import MapContext from "@/utils/map/mapContext";
import { hasOwnProperty } from "@radix-ui/themes"
import TileLayer from "ol/layer/Tile";
import { XYZ } from "ol/source";
import { useContext, useEffect, useState } from "react";
import { Tile, Vector } from "ol/layer";
import {
  Style,
  Text,
  Icon,
  Fill,
  Stroke,
  Circle,
  RegularShape,
} from "ol/style";
import VectorSource from "ol/source/Vector.js";
import Point from "ol/geom/Point.js";
import Feature from "ol/Feature.js";
import { fromLonLat, get } from "ol/proj";

export default function MapTestComponent() {

  const { mapObj } = useContext<any>(MapContext);

  const { map } = mapObj;

  const [isMounted, setIsMounted] = useState<boolean>(false);


  const [nowMap, setNowMap] = useState<string>("vworld");


  useEffect(() => {

    setIsMounted(true);

    console.log(map);


    // vworld base
    const vworld = new TileLayer({

      visible: true,

      properties: { name: "base-vworld" },

      source: new XYZ({

        url: "http://api.vworld.kr/req/wmts/1.0.0/{API 키}/Base/{z}/{y}/{x}.png",

      }),

    });


    // vworld 위성
    const vworldSatelliteLayer = new TileLayer({

      source: new XYZ({

        url: `https://api.vworld.kr/req/wmts/1.0.0/{API 키}/Satellite/{z}/{y}/{x}.jpeg`,

      }),

      properties: { name: "base-vworld-Satellite" },

      minZoom: 5,

      maxZoom: 5,

      zIndex: 10,

      preload: Infinity,

    });

  

    // 마커

    let marker = new Feature({

      geometry: new Point(fromLonLat([128.681882, 35.2279728])),

    });

  

    let myStyle 옴

      map.getLayers().array_.map((m: any) => {

        console.log("m : ", m);

      });

  

      if (nowMap === "vworld") {

        map?.addLayer(vworld);

        map.getAllLayers().map((m: any) => {

          if (m.get("name") && m.get("name") === "base-vworld-Satellite") {

            map.removeLayer(m);

          }

          return;

        });

      } else {

        map?.addLayer(vworldSatelliteLayer);

        map.getAllLayers().map((m: any) => {

          if (m.get("name") && m.get("name") === "base-vworld") {

            map.removeLayer(m);

          }

          return;

        });

      }

  

      if (map.getAllLayers().some((m: any) => m.get("name") === "marker")) {

      } else {

        map.addLayer(markerLayer);

      }

    }

  }, [mapObj, nowMap]);

  

  // 위성지도
  const vworldSatelliteLayer = new TileLayer({

    source: new XYZ({

      url: `https://api.vworld.kr/req/wmts/1.0.0/{API키}/Satellite/{z}/{y}/{x}.jpeg`,

    }),

    properties: { name: "base-vworld-Satellite" },

    minZoom: 5,

    maxZoom: 5,

    zIndex: 10,

    preload: Infinity,

  });


  return (

    <>

      <div

        style={{

          display: "flex",

          flexDirection: "row",

          alignItems: "center",

        }}

      >

        <div style={{ height: "100vh", flexBasis: "30%" }}>

          선택상자

          <button

            style={{ border: "1px solid #dbdbdb", padding: "6px" }}

            onClick={() => {

              if (isMounted) {

                if (nowMap === "vworld") {

                  setNowMap("we");

                } else {

                  setNowMap("vworld");

                }

              }

            }}

          >

            지도모드 전환{" "}

          </button>

        </div>

        <div style={{ height: "100vh", flexBasis: "70%" }}>

          <div

            id="map"

            style={{ position: "relative", width: "100%", height: "100vh" }}

          ></div>

        </div>

      </div>

    </>

  );

}
```