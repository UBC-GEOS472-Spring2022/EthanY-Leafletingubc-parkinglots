# EthanY-Leafletingubc-parkinglots

<!DOCTYPE html>
<html>
  <head>
    <title>Leaflet-ing UBC</title>
    <meta charset="utf-8" />

    <!-- Leaflet styles and code. Place in the <head></head> element. -->
    <link
      rel="stylesheet"
      href="https://unpkg.com/leaflet@1.4.0/dist/leaflet.css"
    />
    <script src="https://unpkg.com/leaflet@1.4.0/dist/leaflet.js"></script>

    <!-- jQuery is a library that simplifies many things in JavaScript. 
	     We'll use it to retrieve data from the web. -->
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>

    <script src="./p5.min.js"></script>
    <!-- this should be unnecessary in this code, as we aren't using p5.js...
       but since I'm showing you the code running in the p5 editor,
       the editor expects p5.js to be here...
       otherwise, it gives you a "TypeError: window.p5 is undefined".
       We won't use any of p5.js's commands, though! -->
  </head>
  <body>
    <div id="mapid" style="width: 600px; height: 400px;"></div>


    <div id="controls" style="margin: 15px;">
      <button id="showAllTreesButton">Show All Parking Lots</button>

      <span style="display:inline-block; width: 30px"></span>
      <input type="text" id="highlightTreesTextEntry" placeholder="FAC_DESCRIPTION" />
      <button id="highlightTreesButton">Highlight</button>
    </div>

    <script>


        var map = L.map('mapid',
                        {
          								center: [49.265637, -123.256113],
          								zoom: 15
        								}  
                  );

      var topoTiles = L.tileLayer(
          								'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
                          {
      										maxZoom: 17,
      										attribution: 'Basemap data: &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Basemap style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)'
      									}
      								).addTo(map);


        var treeMarkerOptions = {
        	radius: 7,
        	fillColor: "#ff007c",
        	color: "#000",
        	weight: 1,
        	opacity: 0.2,
        	fillOpacity: 1
      };

        var treeMarkerHighlightedOptions = {
        	radius: 7,
        	fillColor: "#FF9900",  // Make highlighted trees orange.
        	color: "#000",
        	weight: 1,
        	opacity: 0.3,
        	fillOpacity: 1
      };

        function onEachTree(feature, layer) {

           if (feature.properties && feature.properties.FAC_ID) {
              layer.bindTooltip(feature.properties.FAC_ID);
           }
        }

        function treePointToLayer(feature, latlng) {


          if (taxaToHighlight === "") {
            treeMarkerOptionsToUse = treeMarkerOptions;
          } else {
            treeMarkerOptionsToUse = treeMarkerHighlightedOptions;
          }

          return L.circleMarker(
            latlng,
            treeMarkerOptionsToUse
          );
        }

        function treesToFilter (feature, layer) {
          if(taxaToHighlight === "") {
            return true;
          }
          else if (feature.properties && feature.properties.FAC_DESCRIPTION) {
            return feature.properties.FAC_DESCRIPTION.toLowerCase().includes(taxaToHighlight.toLowerCase());
      		
          } else {
            return false;

          }
        }

      	var lastLayerAdded = {};


        function addTrees() {
          if(map.hasLayer(lastLayerAdded)){  // reset the map layers if relevant.
            map.removeLayer(lastLayerAdded);
          }
          lastLayerAdded = L.geoJSON(treeGeoJSONdata, {
       												pointToLayer: treePointToLayer,
                     				  onEachFeature: onEachTree,
              								filter: treesToFilter
          									 }
                           );
          lastLayerAdded.addTo(map);
        };

        var taxaToHighlight = "";


        function highlightTrees() {
          taxaToHighlight = document
            .getElementById("highlightTreesTextEntry")
            .value;  
          addTrees(); 
          taxaToHighlight = "";  
        }

      var treeGeoJSONdata; 


      $.getJSON("https://raw.githubusercontent.com/UBCGeodata/ubcv-parking/master/geojson/ubcv_parking_www.geojson",
                  function(data){
          					// Store data for later:
      	      		treeGeoJSONdata = data;
          					// Create event listener for ShowAllTrees Button:
      						document
                      .getElementById("showAllTreesButton")
                      .addEventListener("click", addTrees);

          					document
                      .getElementById("highlightTreesButton")
                      .addEventListener("click", highlightTrees);

        					}
        );
    </script>
  </body>
</html>
