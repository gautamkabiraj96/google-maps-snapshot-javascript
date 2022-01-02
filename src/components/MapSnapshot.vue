<template>
  <div>
    <div class="np-header-margins">
      Google Maps Snapshot (<a
        href="https://www.nightprogrammer.com/"
        target="_blank"
        >nightprogrammer.com</a
      >)
      <div>
        <div @click="initiateMapSnapshot" class="np-ib np-snapshot-btn">
          Take snapshot
        </div>
        <div
          @click="clearMapSnapshot"
          style="margin-left: 12px"
          class="np-ib np-snapshot-btn"
          v-if="snapshotImageData"
        >
          Clear snapshot
        </div>
      </div>
    </div>
    <div id="google-map-snapshot" class="np-map-margins"></div>

    <div v-if="snapshotImageData">
      Snapshot:
      <div></div>
      <div>
        <img :src="snapshotImageData" style="height: 400px; width: 600px" />
      </div>
    </div>
  </div>
</template>

<script>
import loadGoogleMapsApi from "load-google-maps-api";
import html2canvas from "html2canvas";
import { gMapsApiKey } from "./../constans";

export default {
  name: "GoogleMapSnapshot",
  data() {
    return {
      map: null,
      snapshotImageData: null,
    };
  },
  mounted() {
    loadGoogleMapsApi({
      key: gMapsApiKey,
      libraries: ["places", "drawing", "geometry"],
    }).then(async () => {
      const mapZoom = 12;
      const { google } = window;
      const mapOptions = {
        zoom: mapZoom,
        mapTypeId: google.maps.MapTypeId.HYBRID,
        center: new google.maps.LatLng({ lat: 23, lng: 57 }),
        mapTypeControl: true,
        streetViewControl: false,
        mapTypeControlOptions: {
          position: google.maps.ControlPosition.BOTTOM_LEFT,
        },
      };
      this.map = new google.maps.Map(
        document.getElementById("google-map-snapshot"),
        mapOptions
      );

      const polygonCoords = [
        { lat: 25.774, lng: -80.19 },
        { lat: 18.466, lng: -66.118 },
        { lat: 32.321, lng: -64.757 },
        { lat: 37.321, lng: -75.757 },
        { lat: 25.774, lng: -80.19 },
      ];

      const tempBounds = new google.maps.LatLngBounds();

      for (let j = 0; j < polygonCoords.length; j++) {
        const x = {
          lat: polygonCoords[j].lat,
          lng: polygonCoords[j].lng,
        };
        const BoundLatLng = new google.maps.LatLng({
          lat: parseFloat(x.lat),
          lng: parseFloat(x.lng),
        });
        tempBounds.extend(BoundLatLng);
      }
      const centroid = tempBounds.getCenter();

      const PolygonShape = new google.maps.Polygon({
        paths: polygonCoords,
        strokeColor: "#ffffff",
        map: this.map,
        strokeWeight: 3,
        fillColor: "#0000ff",
        fillOpacity: 0.2,
        zIndex: 99999,
      });

      PolygonShape.setMap(this.map);
      this.map.setCenter(centroid);
      this.map.setZoom(4);
    });
  },
  methods: {
    clearMapSnapshot() {
      this.snapshotImageData = null;
    },
    initiateMapSnapshot() {
      this.getSnapshotOfElement(
        "#google-map-snapshot",
        0,
        0,
        600,
        400,
        (base64MapData) => {
          this.snapshotImageData = base64MapData;
          alert(
            "Snapshot taken successfully. Scroll down the page to view snapshot."
          );
        }
      );
    },
    getSnapshotOfElement(
      element,
      posX,
      posY,
      width,
      height,
      convertedSnapshotCallback
    ) {
      html2canvas(document.querySelector(element), {
        useCORS: true,
        allowTaint: false,
      }).then((canvas) => {
        const context = canvas.getContext("2d");
        const imageData = context.getImageData(
          posX,
          posY,
          canvas.width,
          canvas.height
        ).data;
        const outputCanvas = document.createElement("canvas");
        const outputContext = outputCanvas.getContext("2d");
        outputCanvas.width = canvas.width;
        outputCanvas.height = canvas.height;

        const outputIData = outputContext.createImageData(
          canvas.width,
          canvas.height
        );
        outputIData.data.set(imageData);
        outputContext.putImageData(outputIData, 0, 0);
        convertedSnapshotCallback(
          //use it to get the raw base64 data without format mention
          //outputCanvas.toDataURL().replace("data:image/png;base64,", "")
          outputCanvas.toDataURL()
        );
      });
    },
  },
};
</script>

<style scoped>
.np-ib {
  display: inline-block;
}
.np-header-margins {
  margin-left: 40px;
  margin-top: 20px;
}

.np-map-margins {
  height: 400px;
  width: 600px;
  margin: 10px 40px;
}

.np-snapshot-btn {
  margin: 14px 0px;
  background: rgb(0, 119, 255);
  width: 140px;
  text-align: center;
  padding: 6px 8px;
  border-radius: 4px;
  color: #ffffff;
  transition: all 0.3s;
  cursor: pointer;
}

.np-snapshot-btn:hover {
  background: rgb(0, 74, 158);
  transition: all 0.3s;
}
</style>