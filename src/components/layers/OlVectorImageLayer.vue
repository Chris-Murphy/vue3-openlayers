<template lang="">
  <div>
    <slot></slot>
  </div>
</template>

<script setup lang="ts">
import { inject, provide, onUnmounted, onMounted, watch, ref } from "vue";

import VectorImageLayer from "ol/layer/VectorImage";
import usePropsAsObjectProperties from "@/composables/usePropsAsObjectProperties";
import {
  layersCommonDefaultProps,
  type LayersCommonProps,
} from "@/components/layers/LayersCommonProps";
import type { Map } from "ol";
import type LayerGroup from "ol/layer/Group";

const props = withDefaults(
  defineProps<
    LayersCommonProps & {
      renderBuffer?: number;
      updateWhileAnimating?: boolean;
      styles?: () => unknown;
      updateWhileInteracting?: boolean;
    }
  >(),
  {
    ...layersCommonDefaultProps,
    renderBuffer: 100,
    updateWhileAnimating: false,
    updateWhileInteracting: false,
  },
);

const map = inject<Map>("map");
const layerGroup = inject<LayerGroup | null>("layerGroup", null);

const { properties } = usePropsAsObjectProperties(props);

const vectorImageLayer = ref(new VectorImageLayer(properties));

watch(properties, () => {
  vectorImageLayer.value.setProperties(properties);
});

watch(
  () => props.opacity,
  (newOpacity: number) => {
    vectorImageLayer.value.setOpacity(newOpacity);
  },
  { immediate: true },
);
watch(
  () => props.visible,
  (newVisible: boolean) => {
    vectorImageLayer.value.setVisible(newVisible);
  },
  { immediate: true },
);
  
onMounted(() => {
  map?.addLayer(vectorImageLayer.value);

  if (layerGroup) {
    const layerCollection = layerGroup.getLayers();
    layerCollection.push(vectorImageLayer.value);
    layerGroup.setLayers(layerCollection);
  }
});

onUnmounted(() => {
  map?.removeLayer(vectorImageLayer.value);
});

provide("vectorLayer", vectorImageLayer);
provide("stylable", vectorImageLayer);

defineExpose({
  vectorImageLayer,
});
</script>
