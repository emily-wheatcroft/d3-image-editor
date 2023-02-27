<template>
  <div>
    <div class="template">
      <div id="resizable" :class="mode">
        <img
          id="image"
          src="https://media-manager.noticiasaominuto.com/1920/naom_634bc7b9686c2.jpg?crop_params=eyJsYW5kc2NhcGUiOnsiY3JvcFdpZHRoIjoyNTYyLCJjcm9wSGVpZ2h0IjoxNDQxLCJjcm9wWCI6LTEsImNyb3BZIjoxNDN9fQ=="
        />
        <div v-show="mode === `resize`" class="resizer resizer-left"></div>
        <div v-show="mode === `resize`" class="resizer resizer-right"></div>
        <div v-show="mode === `resize`" class="resizer resizer-top"></div>
        <div v-show="mode === `resize`" class="resizer resizer-bottom"></div>
      </div>
    </div>

    <div class="options-bar">
      <select v-model="mode">
        <option value="resize">Resize</option>
        <option value="crop">Crop</option>
      </select>

      <button v-if="mode === `crop`" @click="undoCrop">Undo</button>
    </div>
  </div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "HelloWorld",
  props: {},
  data: () => ({
    mode: `resize`,
  }),
  mounted() {
    this.setResize();
  },
  watch: {
    mode(value) {
      if (value === `resize`) return this.setResize();
      if (value === `crop`) return this.setCrop();
    },
  },
  methods: {
    setResize() {
      let resizable = d3.select("#resizable.resize");
      let resizerRight = resizable.select(".resizer-right");
      let resizerLeft = resizable.select(".resizer-left");
      let resizerBottom = resizable.select(".resizer-bottom");
      let resizerTop = resizable.select(".resizer-top");
      let img = resizable.select("img");

      let dragResizeRight = d3.drag().on("drag", function (e) {
        // Determine resizer position relative to resizable (parent)
        let x = d3.pointer(e, this.parentNode)[0];

        // Avoid negative or really small widths
        x = Math.max(50, x);

        resizable.style("width", x + "px");
      });

      let dragResizeBottom = d3.drag().on("drag", function (e) {
        // Determine resizer position relative to resizable (parent)
        let y = d3.pointer(e, this.parentNode)[1];

        // Avoid negative or really small widths
        y = Math.max(50, y);

        resizable.style("height", y + "px");
      });

      let dragResizeLeft = d3.drag().on("drag", function (e) {
        let imageDimensions = resizable._groups[0][0].getBoundingClientRect();

        // Determine resizer position relative to resizable (parent)
        let x = d3.pointer(e, resizable._groups[0].parentNode)[0];
        let width = imageDimensions.width + (imageDimensions.left - x);
        resizable.style("left", `${x - 8}px`);
        resizable.style("width", `${Math.max(width, 50)}px`);
      });

      let dragResizeTop = d3.drag().on("drag", function (e) {
        let imageDimensions = resizable._groups[0][0].getBoundingClientRect();

        // Determine resizer position relative to resizable (parent)
        let y = d3.pointer(e, resizable._groups[0].parentNode)[1];
        let height = imageDimensions.height + (imageDimensions.top - y);
        resizable.style("top", `${y - 8}px`);
        resizable.style("height", `${Math.max(height, 50)}px`);
      });

      let imageOriginal = [0, 0];

      let dragImage = d3
        .drag()
        .on("start", function (e) {
          console.log('goingggg')
          imageOriginal = d3.pointer(e, resizable._groups[0][0]);
        })
        .on("drag", function (e) {
          let [x, y] = d3.pointer(e, resizable._groups[0].parentNode);
          resizable.style("left", `${x - imageOriginal[0]}px`);
          resizable.style("top", `${y - imageOriginal[1]}px`);
        });

      resizerRight.call(dragResizeRight);
      resizerBottom.call(dragResizeBottom);
      resizerLeft.call(dragResizeLeft);
      resizerTop.call(dragResizeTop);
      console.log("switching back");
      img.call(dragImage);
    },
    setCrop() {
      let resizable = d3.select("#resizable.resize");
      let img = resizable.select("img");
      let imageDimensions = resizable._groups[0][0].getBoundingClientRect();
      let startX, startY, currentX, currentY;
      let dragImage = d3
        .drag()
        .on("start", function (e) {
          startX = e.x;
          startY = e.y;
        })
        .on("drag", function (e) {
          currentX = e.x;
          currentY = e.y;
          document.getElementById(`image`).style[
            `clip-path`
          ] = `inset(${startY}px ${imageDimensions.width - currentX}px ${
            imageDimensions.height - currentY
          }px ${startX}px)`;
        });
      img.call(dragImage);
    },
    undoCrop() {
      document.getElementById(`image`).style[`clip-path`] = `unset`;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.template {
  width: 600px;
  height: 400px;
  background-color: #1e1e1e;
  overflow: hidden;
}

#resizable {
  position: relative;
  width: auto;
  height: auto;
}

#resizable.crop {
  cursor: crosshair;
}

#resizable img {
  max-width: 100%;
  max-height: 100%;
  overflow: hidden;
}

#resizable .resizer,
.cropper {
  position: absolute;
  opacity: 0.5;
  cursor: pointer;
}

#resizable .resizer {
  background: orange;
}

#resizable .cropper {
  background: #eb4755;
}

.resizer-left,
.resizer-right,
.cropper-left,
.cropper-right {
  width: 10px;
}

.resizer-top,
.resizer-bottom,
.cropper-top,
.cropper-bottom {
  height: 10px;
}

.resizer-right,
.cropper-right {
  top: 0;
  bottom: 0;
  right: -5px;
}

.resizer-left,
.cropper-left {
  top: 0;
  bottom: 0;
  left: -5px;
}

.resizer-top,
.cropper-top {
  top: -5px;
  bottom: calc(100% - 5px);
  left: 0;
  right: 0;
}

.resizer-bottom,
.cropper-bottom {
  top: calc(100% - 5px);
  bottom: -5px;
  left: 0;
  right: 0;
}

.options-bar {
  display: flex;
}
</style>
