<template>
  <div
    class="time-block"
    :class="['disabled', 'default', 'active'][pattern]"
    @click="pattern && emit('click', [colNumber, gridNumber])"
  ></div>
</template>

<script setup>
import { inject, ref, watch, onMounted } from "vue";
const props = defineProps({
  text: {
    type: String,
  },
  colNumber: {
    type: Number,
  },
  gridNumber: {
    type: Number,
  },
  sn: {
    type: Number,
  },
  timeSpan: {
    type: Array,
  },
});

const emit = defineEmits(["click"]);
const eventBus = inject("eventBus");
const disabledSN = inject("disabledSNs");
const pattern = ref(1); // 0: disabled  1: default  2: active

watch(disabledSN, () => {
  pattern.value = +(!disabledSN.value.includes(props.sn));
})

eventBus.listen("update", (renderMsg) => {
  const { border, actived } = renderMsg;
  const sn = props.sn;

  // 没有选择时，重置状态
  if (actived.length === 0) {
    pattern.value = +(!disabledSN.value.includes(props.sn));
    return;
  }

  // 可选范围(0, 1)
  pattern.value = +border.includes(sn);
  // 已选范围(2)
  actived.includes(sn) && (pattern.value = 2);

  disabledSN.value.includes(sn) && (pattern.value = 0);
});
</script>

<style scoped>
.time-block {
  width: 48px;
  height: 25px;
  text-align: center;
  line-height: 25px;
  cursor: pointer;
}
.time-block.default {
  background-color: #e3f2fd;
}
.time-block.active {
  background-color: #2196f3;
}
.time-block.disabled {
  cursor: not-allowed;
  background-image: linear-gradient(
    -45deg,
    #f5f5f5,
    #f5f5f5 40%,
    #e0e0e0 0,
    #e0e0e0 50%,
    #f5f5f5 0,
    #f5f5f5 90%,
    #e0e0e0 0,
    #e0e0e0
  );
  background-size: 6px 6px;
  background-repeat: repeat;
}
</style>
