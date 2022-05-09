<template>
  <div class="polygonjs-scene-container">
    <div class="polygonjs-scene" ref="mountpoint"></div>
    <div
      v-if="displayLoadingProgressBar"
      class="progress-bar"
      :class="progressBarClassObject"
    >
      <div class="progress-bar-bar" :style="progressBarBarStyleObject"></div>
    </div>
  </div>
</template>

<script>
import { PolyEventName } from "@polygonjs/polygonjs/dist/src/engine/io/common/EventsDispatcher";

export default {
  name: "Polygonjs-Scene",
  props: {
    // sceneName: String,
    displayLoadingProgressBar: {
      type: Boolean,
      default: true,
    },
    printWarnings: {
      type: Boolean,
      default: false,
    },
    loadFunction: {
      type: Function,
    },
  },
  data: function () {
    return {
      progress: 0,
      scene: null,
      viewer: null,
    };
  },
  mounted: async function () {
    const domElement = this.$refs.mountpoint;
    const { loadFunction } = this.$props;

    const loadedData = await loadFunction({
      onProgress: (progress) => {
        this.$data.progress = progress;
        this.$emit("progress", progress);
      },
      domElement,
      printWarnings: this.$props.printWarnings,
    });
    const { scene, viewer } = loadedData;
    this.$data.scene = scene;
    this.$data.viewer = viewer;
    const sceneReady = PolyEventName.SCENE_READY.toLowerCase().replace(
      "poly",
      ""
    );
    const viewerMounted = PolyEventName.VIEWER_MOUNTED.toLowerCase().replace(
      "poly",
      ""
    );
    this.$emit(sceneReady, scene);
    this.$emit(viewerMounted, viewer);
  },
  beforeDestroy: function () {
    const scene = this.$data.scene;
    if (scene) {
      scene.dispose();
    }
    const viewer = this.$data.viewer;
    if (viewer) {
      viewer.dispose();
    }
  },
  computed: {
    progressBarClassObject: function () {
      const visible = this.$data.progress > 0 && this.$data.progress < 1;
      return {
        fadeable: true,
        visible: true,
        hidden: !visible,
      };
    },
    progressBarBarStyleObject: function () {
      const percent = Math.round(this.$data.progress * 100);
      return {
        width: `${percent}%`,
      };
    },
  },
};
</script>

<style scoped>
.polygonjs-scene-container {
  position: relative;
}
.polygonjs-scene-container,
.polygonjs-scene {
  height: 100%;
}
.polygonjs-scene canvas {
  display: block;
}
/*
progress bar
*/
.progress-bar {
  position: absolute;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 4px;
  pointer-events: none;
}
.progress-bar-bar {
  height: 100%;
  background-color: orange;
  animation-name: progress-bar-color;
  animation-duration: 4s;
  animation-iteration-count: infinite;
}
.fadeable {
  opacity: 1;
  transition: opacity 0.5s ease-in-out;
}
.visible {
  opacity: 1;
}
.hidden {
  opacity: 0;
}
@keyframes progress-bar-color {
  0% {
    background-color: rgb(115, 214, 115);
  }
  50% {
    background-color: rgb(60, 60, 211);
  }
  100% {
    background-color: rgb(115, 214, 115);
  }
}
</style>
