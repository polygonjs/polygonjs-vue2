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

<script lang="ts">
import { PolyEventName } from "@polygonjs/polygonjs/dist/src/engine/io/common/EventsDispatcher";
import { PolyScene } from "@polygonjs/polygonjs/dist/src/engine/scene/PolyScene";

import {
  defineComponent,
  Ref,
  ref,
  computed,
  onMounted,
  onBeforeUnmount,
} from "@vue/composition-api";
import { Context } from "mocha";
import { BaseViewerType } from "@polygonjs/polygonjs/dist/src/engine/viewers/_Base";

interface LoadSceneOptions {
  onProgress: (progress: number) => void;
  domElement: HTMLElement;
  printWarnings: boolean;
}
interface LoadedData {
  scene: PolyScene;
  viewer: BaseViewerType | undefined;
}
type LoadScene = (options: LoadSceneOptions) => Promise<LoadedData>;
interface PolySceneProps {
  displayLoadingProgressBar: boolean;
  printWarnings: boolean;
  loadFunction: LoadScene;
}

export default defineComponent({
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
  set(props: PolySceneProps, context: Context) {
    const progress = ref(0);
    const mountPoint: Ref<HTMLElement | null> = ref(null);
    let scene: PolyScene | undefined;
    let viewer: BaseViewerType | undefined;

    onMounted(_loadSceneAndViewer);

    onBeforeUnmount(_disposeSceneAndViewer);

    async function _loadSceneAndViewer() {
      const domElement = mountPoint.value;
      if (!domElement) {
        return;
      }
      const { loadFunction } = props;
      const loadedData = await loadFunction({
        onProgress: (p) => {
          progress.value = p;
          context.emit("progress", p);
        },
        domElement,
        printWarnings: props.printWarnings,
      });
      scene = loadedData.scene;
      viewer = loadedData.viewer;
      const sceneReady = PolyEventName.SCENE_READY.toLowerCase().replace(
        "poly",
        ""
      );
      const viewerMounted = PolyEventName.VIEWER_MOUNTED.toLowerCase().replace(
        "poly",
        ""
      );
      context.emit(sceneReady, scene);
      context.emit(viewerMounted, viewer);
    }

    function _disposeSceneAndViewer() {
      scene?.dispose();
      viewer?.dispose();
    }

    const progressBarClassObject = computed(() => {
      const visible = progress.value > 0 && progress.value < 1;
      return {
        fadeable: true,
        visible: true,
        hidden: !visible,
      };
    });
    const progressBarBarStyleObject = computed(() => {
      const percent = Math.round(progress.value * 100);
      return {
        width: `${percent}%`,
      };
    });

    return { mountPoint, progressBarClassObject, progressBarBarStyleObject };
  },
});
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
