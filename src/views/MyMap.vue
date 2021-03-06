
<template>
  <div class="MyMap">
    <h1>Hello Map</h1>
    <div id="mapContainer" class="basemap"></div>
    <div id="spotlight" class="light"></div>
    <div id="story">
      <div id="character"></div>

      <p id="storytext"></p>
    </div>
    <li v-for="commute in commutes" :key="commute.fields">
      <Map
        :person="commute.fields.name"
        :img="commute.fields.picture.fields.file.url"
        :location="commute.fields.whereAmI.lat"
      />
    </li>
  </div>
</template>


<script defer>
let storypart = 0;
let imgCounter = 0; //is used to switch between two images
let character = document.getElementById("character");
let storyEnd = true;
let storyCounter = 0;
let moveflag = true;

let fullText;
let textElements;
let textField;
let imgs;

// @ is an alias to /src
import Map from "@/components/Map.vue";
import contentfulClient from "@/modules/contentful.js";
import mapboxgl from "mapbox-gl";
import getCoordinatesFromGpxFile from "@/modules/gpx-utilities.js";

export default {
  name: "MyMap",
  components: {
    Map,
  },
  data: function () {
    return {
      accessToken:
        "pk.eyJ1IjoiamEtbmVpbiIsImEiOiJja2gzZTg3cHExMzcwMnVvMzE5cWtyb2czIn0.nEY6G_mQjFnai5glV1G2SQ",
      commutes: [],
    };
  },

  //================================================
  // Loading Map
  //================================================
  created: async function () {
    console.log("created a module MyMap");
    let result = await contentfulClient.getEntries({ content_type: "person" });
    this.commutes = result.items;
  },

  mounted: function () {
    mapboxgl.accessToken = this.accessToken;

    var map = new mapboxgl.Map({
      container: "mapContainer",
      style: "mapbox://styles/ja-nein/ckhen3exh0tvv19p4f52e76c9",
      center: [8.286813, 47.082362], //   8.286813, 47.082362
      zoom: 15.7,
      maxBounds: [
        [8.277487, 47.073149], //47.073149, 8.277487
        [8.30644, 47.088459], //47.094393, 8.307152
      ],
    });

    //===========================================
    //Creating GPX Trail
    //===========================================
    // Displaying a GPX track
    map.on("load", async function () {
      let result = await contentfulClient.getEntries({
        content_type: "gpx",
      });

      let coordinates = await getCoordinatesFromGpxFile(
        result.items[0].fields.gpxFile.fields.file.url // a link to you gpx file in Contentful
      );

      map.addSource("route", {
        type: "geojson",
        data: {
          type: "Feature",
          geometry: {
            type: "LineString",
            coordinates: coordinates,
          },
        },
      });
      map.addLayer({
        id: "route",
        type: "line",
        source: "route",
        layout: {
          "line-join": "round",
          "line-cap": "round",
        },
        paint: {
          "line-color": "#E1BC29",
          "line-width": 5,
        },
      });

      //===========================================
      //Creating Markers
      //===========================================
      let markers = [];
      //let marker;

      result = await contentfulClient.getEntries({
        content_type: "storyline",
      });

      //===========================================
      //geojson markers from mapbox tutorial
      //===========================================
      //geojson from mapbox tutorial
      var geojson = {
        type: "FeatureCollection",
        features: [],
      };

      //getting the coordinates from the contentful items and adding obj to geojson.
      function addingCoordinates() {
        result.items.forEach((element) => {
          let obj = {
            type: "Feature",
            geometry: {
              type: "Point",
              coordinates: [
                element.fields.location.lon,
                element.fields.location.lat,
              ],
            },
            properties: {
              title: "Mapbox",
              description: "Center Map",
            },
          };
          geojson.features.push(obj);
        });

        // add markers to map
        geojson.features.forEach(function (marker) {
          let mark;
          let index = geojson.features.indexOf(marker, 0);
          let url =
            "url(http:" + result.items[index].fields.icon.fields.file.url + ")";

          // create a HTML element for each feature
          var el = document.createElement("div");
          el.className = "marker";
          el.style.width = "var(--markerSize)";
          el.style.height = "var(--markerSize)";
          el.style.backgroundSize = "var(--markerSize)";
          el.style.backgroundImage = url;
          el.style.backgroundRepeat = "no-repeat";
          el.style.borderRadius = "50%";
          el.style.position = "absolute";
          el.style.zIndex = "4";
          el.style.borderStyle = "solid";
          el.style.borderWidth = "var(--borderWidth)";
          el.style.borderColor = "var(--border)";

          if (index == 0) {
            let popup = new mapboxgl.Popup({ closeOnClick: false })
              .setHTML(
                '<h1 style="color:var(--markedText); font-weight: bold;">Start Here!</h1>' +
                  '<p style="color:var(--markedText)">and follow the path</p>'
              )
              .setLngLat([
                result.items[1].fields.location.lon,
                result.items[1].fields.location.lat,
              ]);
            popup.addTo(map);
            popup.getElement().style.fontWeight = "bold";

            // make a marker for each feature and add to the map
            mark = new mapboxgl.Marker(el).setLngLat(
              marker.geometry.coordinates
            );
          } else {
            // make a marker for each feature and add to the map
            mark = new mapboxgl.Marker(el).setLngLat(
              marker.geometry.coordinates
            );
          }

          // mark.getElement().style.backgroundImage = url;
          mark.getElement().style.zIndex = "15";
          mark.getElement().style.backgroundColor = "black";
          mark.addTo(map);
          markers[index] = mark;
          //--------------------------------------------------------------------- Event Listeners
          //adding event listener right away
          //----------------------------------------------------
          markers[index]
            .getElement()
            .addEventListener("mouseleave", function () {
              markerNormal(index);
            });
          markers[index].getElement().addEventListener("click", function () {
            changeView(index);
          });
          markers[index]
            .getElement()
            .addEventListener("mouseenter", function () {
              markerHover(index);
            });
        });
      }
      await addingCoordinates();

      /*
     //use this to make a separate marker that is absolutely intependent of the others.
let purrlock;
let purrlockmark;
     //this in adding coordinates method
            purrlock = {
          type: "FeatureCollection",
          features: [
            {
              type: "Feature",
              geometry: {
                type: "Point",
                coordinates: [
                  8.286813, 47.082362
                ],
              },
              properties: {
                title: "Mapbox",
                description: "Center Map",
              },
            },
          ],
        };
      await purrlockMarker();

      function purrlockMarker() {
        let mark;
        let url =
          "url(http:" + result.items[1].fields.icon.fields.file.url + ")";

        // create a HTML element for each feature
        var el = document.createElement("div");
        el.className = "marker";
        el.style.width = "var(--markerSize)";
        el.style.height = "var(--markerSize)";
        el.style.backgroundSize = "var(--markerSize)";
        el.style.backgroundImage = url;
        el.style.backgroundRepeat = "no-repeat";
        el.style.backgroundColor = "transparent";
        el.style.borderRadius = "50%";
        el.style.position = "absolute";
        el.style.zIndex = "4";

          // make a marker for each feature and add to the map
          mark = new mapboxgl.Marker(el).setLngLat(
            purrlock.features[0].geometry.coordinates
          );
          // make a marker for each feature and add to the map
          mark = new mapboxgl.Marker(el).setLngLat(
            purrlock.features[0].geometry.coordinates
          );
        

        // mark.getElement().style.backgroundImage = url;
        mark.getElement().style.zIndex = "15";
        mark.getElement().style.backgroundColor = "black";
        mark.addTo(map);
        purrlockmark = mark;
      console.log(purrlockmark);
      }
     */
      //console.log(result.items[0].fields.location.lat, result.items[0].fields.location.lon)

      //   map.fire("click", [8.2900136403198, 47.082168941708 ]);

      //===========================================
      //My marker methods
      //===========================================
      //changes marker when hovering over it
      function markerHover(i) {
        markers[i].getElement().style.boxShadow =
          "0px 0px var(--shadowWidth) var(--markerShadow),0px 0px var(--shadowWidth) var(--markerShadow),0px 0px var(--shadowWidth) var(--markerShadow),0px 0px var(--shadowWidth) var(--markerShadow)";
      }

      //-------------------------------
      //normal marker funktion
      //removing box shadow when nothing is hovering over it
      function markerNormal(i) {
        markers[i].getElement().style.boxShadow = "unset";
      }

      //===========================================
      //Story Methods
      //===========================================
      //Adding Elements to story div
      function changeView(i) {
        storyEnd = false;
        //console.log(i, storypart);
        if (i === storypart) {
          character = document.getElementById("character");
          moveflag = false;
          character.style.zIndex = "25";
          let burl =
            "url( 'http:" +
            result.items[i].fields.background.fields.file.url +
            "')";
          document.getElementById("story").style.backgroundImage = burl;
          document.getElementById("story").style.zIndex = "20";

         

          storyTelling(i);
        } else {
          console.log("wrong story part");
        }
      }

      function storyEvent() {
        if (storyEnd == true) {
          storyCounter = 0;
        } else {
          storyTelling();
        }
      }
            document.getElementById("character").addEventListener("click", () => storyEvent());

      changeView(0);


      function storyTelling() {
        let story = document.getElementById("story");
        textField = document.getElementById("storytext");

        if (storyCounter === 0) {
          //splitting text into seperate Elements
          if (storyCounter === 0 || storyCounter == result.items.length - 1) {
            character.setAttribute("class", "intro");
            story.setAttribute("class", "intro");
            textField.setAttribute("class", "intro");
          }
          fullText =
            result.items[storypart].fields.text.content[0].content[0].value;
          textElements = fullText.split("@");
          //getting images from contentful
          imgs = result.items[storypart].fields.img;

          character = document.getElementById("character");
          textField.style.zIndex = "30";
        }
        textField.innerHTML = textElements[storyCounter];

        if (textElements.length === imgs.length) {
          let url = "url(" + imgs[storyCounter].fields.file.url + ")";
          character.style.backgroundImage = url;
        } else {
          if (imgCounter <= 1 || textElements.length - 1 == storyCounter) {
            let url = "url( http:" + imgs[0].fields.file.url + ")";
            //changing classes
            if (storypart != 0 && storyCounter != result.items.length) {
              character.setAttribute("class", "purrlock story" + storypart);
              story.setAttribute("class", "purrlock story" + storypart);
              textField.setAttribute("class", "purrlock story" + storypart);
            }
            character.style.backgroundImage = url;
            imgCounter++;
          } else if (imgCounter == 2) {
            if (storypart != 0 && storyCounter != result.items.length - 1) {
              character.setAttribute(
                "class",
                "other " + imgs[1].fields.file.fileName + " story" + storypart
              );
              story.setAttribute(
                "class",
                "other " + imgs[1].fields.file.fileName + " story" + storypart
              );
              textField.setAttribute(
                "class",
                "other " + imgs[1].fields.file.fileName + " story" + storypart
              );
            }
            let url = "url( 'http:" + imgs[1].fields.file.url + "')";
            character.style.backgroundImage = url;
            imgCounter--;
          }
        }
        if (storyCounter >= textElements.length) {
          //console.log("end of story part");
          storyEnd = true;
          storyCounter = 0;
          imgCounter = 0;
          character.style.zIndex = "0";
          document.getElementById("story").style.zIndex = "0";
          textField.style.zIndex = "0";
          //changes the map marker icon to the crossen one.
          let url =
            "url('http:" +
            result.items[storypart].fields.crossedIcons.fields.file.url +
            "')";
          markers[storypart].getElement().style.backgroundImage = url;
          storypart++;
          moveflag = true;
          if (storypart >= result.items.length) {
            storypart = 0;
            changeView(storypart);
          }
        } else {
          storyCounter += 1;
        }
      }
    });
  },
};

//===========================================
//Spotlight
//===========================================
window.addEventListener("mousemove", (e) => {
  if (moveflag == true) {
    spotlightMove(e);
  }
});
function spotlightMove(e) {
  let string =
    "radial-gradient(circle at " +
    (e.pageX / window.innerWidth) * 100 +
    "% " +
    (e.pageY / window.innerHeight) * 100 +
    "%,transparent var(--spotlightTransparent),var(--spotlightColor) var(--spotlightSize))";
  document.getElementById("spotlight").style.backgroundImage = string;
}
</script>


<style scoped>
@import url("https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css");

#mapContainer {
  position: fixed;
  top: 0px;
  left: 0px;
  z-index: 1;
  height: 100%;
  width: 100%;
}

#spotlight {
  z-index: 5;
  position: absolute;
  background-image: radial-gradient(
    circle at 20% 20%,
    transparent var(--spotlightTransparent),
    var(--spotlightColor) var(--spotlightSize)
  );
  height: 100%;
  width: 100%;
  top: 0px;
  left: 0px;
  pointer-events: none;
}

#route {
  z-index: 10;
}

.marker {
  background-size: cover;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
  z-index: 4;
}

.mapboxgl-marker {
  width: 25px;
  height: 25px;
  border-radius: 50%;
}

.mapboxgl-popup {
  max-width: 200px;
}

#character {
  background-color: transparent;
  background-repeat: no-repeat;
  background-position: bottom right;
  height: 100%;
  width: 100%;
  background-size: 50%;
  bottom: 0%;
  right: 0%;
  position: absolute;
  z-index: 0;
  background-image: unset;
}

#character.other.story1 {
    background-position: bottom right;

}

#character.other {
  background-position: bottom left;
}

#character.other.story3 {
  background-position: bottom right;
}

#story {
  overflow: hidden;
  background-color: var(--background);
  background-position: center;
  background-size: auto 100%;
  overflow: auto;
  background-repeat: no-repeat;
  z-index: 0;
  height: 100%;
  width: 100%;
  position: absolute;
  top: 0px;
  left: 0px;
}

#storytext {
  pointer-events: none;
  font-family: monospace, Helvetica, Arial, sans-serif;
  z-index: 0;
  display: block;
  position: absolute;
  width: 30%;
  top: 20%;
  right: 35%;
  text-align: center;
  padding: 20px;
  border: solid black;
  border-width: 1px;
  background-color: white;
  color: black;
}

#storytext.intro {
  font-family: FilmNoir;
  color: white;
  background-color: black;
  font-size: 3em;
  width: 100%;
  top: 30%;
  left: 0;
  animation: scaleup 0.5s ease-in-out forwards; /* transition: 0.5s all; */
}

#storytext.story5 {
  font-family: FilmNoir;
  color: white;
  background-color: transparent;
  font-size: 3em;
  width: 100%;
  top: 30%;
  left: 0;
  animation: scaleup 0.5s ease-in-out forwards;  
}

@keyframes scaleup {
  0%{transform: scale(0);}
  100%{transform: scale(1);}
  
}
</style>



<!--
console.log(backgrounddiv);
          backgrounddiv.style.height = "100%";
          backgrounddiv.style.width = "100%";
          backgrounddiv.style.backgroundColor = "black";
          backgrounddiv.style.backgroundPosition = "bottom left";
          backgrounddiv.style.zIndex = "20";
          backgrounddiv.style.position = "absolute";
          backgrounddiv.style.backgroundRepeat = "no-repeat";
          backgrounddiv.style.top = "0px";
          backgrounddiv.style.left = "0px";
          console.log("story time");
-->
