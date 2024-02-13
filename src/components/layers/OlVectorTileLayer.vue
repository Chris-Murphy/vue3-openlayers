<template>
  <div>
    <slot></slot>
  </div>
</template>

<script setup lang="ts">
import { inject, provide, onUnmounted, onMounted, watch, ref } from "vue";
import VectorTileLayer, {
  type VectorTileRenderType,
} from "ol/layer/VectorTile";
import type Map from "ol/Map";
import usePropsAsObjectProperties from "@/composables/usePropsAsObjectProperties";
import {
  layersCommonDefaultProps,
  type LayersCommonProps,
} from "@/components/layers/LayersCommonProps";
import type { StyleLike } from "ol/style/Style";
import type { FlatStyleLike } from "ol/style/flat";
import type LayerGroup from "ol/layer/Group";

const props = withDefaults(
  defineProps<
    LayersCommonProps & {
      renderBuffer?: number;
      updateWhileAnimating?: boolean;
      styles?: StyleLike | FlatStyleLike | null;
      updateWhileInteracting?: boolean;
      renderMode?: VectorTileRenderType;
    }
  >(),
  {
    ...layersCommonDefaultProps,
    renderBuffer: 100,
    updateWhileAnimating: false,
    updateWhileInteracting: false,
    renderMode: "hybrid",
  },
);

const map = inject<Map>("map");
const layerGroup = inject<LayerGroup | null>("layerGroup", null);

const { properties } = usePropsAsObjectProperties(props);

const vectorTileLayer = ref(new VectorTileLayer(properties));

watch(properties, () => {
  vectorTileLayer.value.setProperties(properties);
});

watch(
  () => props.opacity,
  (newOpacity: number) => {
    vectorTileLayer.value.setOpacity(newOpacity);
  },
  { immediate: true },
);
watch(
  () => props.visible,
  (newVisible: boolean) => {
    vectorTileLayer.value.setVisible(newVisible);
  },
  { immediate: true },
);
  
onMounted(() => {
  map?.addLayer(vectorTileLayer.value);
  if (layerGroup) {
    const layerCollection = layerGroup.getLayers();
    layerCollection.push(vectorTileLayer.value);
    layerGroup.setLayers(layerCollection);
  }
});

onUnmounted(() => {
  map?.removeLayer(vectorTileLayer.value);
});

provide("vectorTileLayer", vectorTileLayer);
provide("stylable", vectorTileLayer);

defineExpose({
  vectorTileLayer,
});
</script>
