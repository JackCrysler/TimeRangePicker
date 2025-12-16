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
}
.time-block.default {
  background-color: #cedceb;
}
.time-block.active {
  background-color: #008bd6;
}
.time-block.disabled {
  background-image: linear-gradient(
    -45deg,
    #f2f2f2,
    #f2f2f2 40%,
    #e8e8e8 0,
    #e8e8e8 50%,
    #f2f2f2 0,
    #f2f2f2 90%,
    #e8e8e8 0,
    #e8e8e8
  );
  background-size: 4px 4px;
  background-repeat: repeat;
}
</style>
