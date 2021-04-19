<template>
  <div id="app">
    <div id="map"></div>
    <table>
      <tr>
        <th>id</th>
        <th>pontos</th>
        <th>opções</th>
      </tr>
      <tr v-for="f in fig" :key="f.idElement">
        <td>{{ f.idElement }}</td>
        <td>{{ f.getPath().length }}</td>
        <td>
          <button @click="remove(f.idElement)">delete</button
          ><button @click="edit(f.idElement)">editar</button>
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      map: {},
      trig: {
        ll: [
          { lat: -19.7531905, lng: -47.9369182 },
          { lat: -19.7541705, lng: -47.9369182 },
          { lat: -19.7531905, lng: -47.9339482 },
        ],
      },
      fig: [],
    };
  },
  mounted() {
    var script = document.createElement("script");
    script.setAttribute("id", "mapScript");
    script.src =
      "https://maps.googleapis.com/maps/api/js?key=AIzaSyBe5g9pdbE_5T0F2MdPMdu6FkiSfdBTuwg&callback=initMap&libraries=drawing";
    script.async = true;
    window.initMap = () => {
      this.map = new window.google.maps.Map(document.getElementById("map"), {
        center: { lat: -19.7531905, lng: -47.9369182 },
        zoom: 17,
      });
      const drawingManager = new window.google.maps.drawing.DrawingManager({
        drawingMode: null,
        drawingControl: true,
        drawingControlOptions: {
          position: window.google.maps.ControlPosition.TOP_CENTER,
          drawingModes: [
            // window.google.maps.drawing.OverlayType.MARKER,
            // window.google.maps.drawing.OverlayType.CIRCLE,
            window.google.maps.drawing.OverlayType.POLYGON,
            // window.google.maps.drawing.OverlayType.POLYLINE,
            // window.google.maps.drawing.OverlayType.RECTANGLE,
          ],
        },
        polygonOptions: {
          strokeColor: "#FF0000",
          strokeOpacity: 0.8,
          strokeWeight: 2,
          fillColor: "#FF0000",
          fillOpacity: 0.35,
          editable: false,
        },
      });
      window.google.maps.event.addListener(
        drawingManager,
        "polygoncomplete",
        (poli) => {
          poli["idElement"] = this.fig.length;
          poli.addListener("rightclick", (ev) =>
            this.click(ev, poli["idElement"])
          );
          this.fig.push(poli);
        }
      );
      drawingManager.setMap(this.map);
      this.trig = new window.google.maps.Polygon({
        paths: this.trig.ll,
        strokeColor: "#FF0000",
        strokeOpacity: 0.8,
        strokeWeight: 2,
        fillColor: "#FF0000",
        fillOpacity: 0.35,
        editable: false,
      });
      this.trig["idElement"] = this.fig.length;
      this.trig.setMap(this.map);
      this.trig.addListener("rightclick", (ev) =>
        this.click(ev, this.trig["idElement"])
      );
      this.fig.push(this.trig);
    };
    document.head.appendChild(script);
  },
  methods: {
    remove(id) {
      this.fig[id].setMap(null);
      this.fig.splice(id, 1);
    },
    edit(id) {
      this.fig[id].setEditable(!this.fig[id].getEditable());
    },
    click(ev, id) {
      if (this.fig[id].getEditable()) {
        const vertices = this.fig[id].getPath();
        if (vertices.length > 3) {
          for (let i = 0; i < vertices.getLength(); i++) {
            const xy = vertices.getAt(i);
            if (ev.latLng == xy) {
              this.fig[id].getPath().removeAt(i);
            }
          }
        }
      }
      // this.fig[id].setEditable(!this.fig[id].getEditable());
      // const vertices = this.trig.getPath();
      // let contentString =
      //   "Clicked location: \n" + ev.latLng.lat() + "," + ev.latLng.lng() + "\n";
      // // Iterate over the vertices.
      // for (let i = 0; i < vertices.getLength(); i++) {
      //   const xy = vertices.getAt(i);
      //   contentString +=
      //     "\n" + "Coordinate " + i + ":\n" + xy.lat() + "," + xy.lng();
      // }
      // console.log(contentString);
    },
  },
};
</script>

<style>
#map {
  height: 400px;
}
/* Optional: Makes the sample page fill the window. */
html,
body,
#app {
  height: 100%;
  margin: 0;
  padding: 0;
}
</style>