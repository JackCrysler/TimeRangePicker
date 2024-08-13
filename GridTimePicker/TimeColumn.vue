<template>
  <view class="time-column">
    <view class="plain">{{ displayTime }}</view>
    <TimeBlock
      v-for="(x, y) in columnInfo"
      :col-number="colNumber"
      :grid-number="x.colIndex"
      :key="x.sn"
      :time-span="x.timeSpan"
      :sn="x.sn"
      @click="(position) => handleClick(position, x.sn)"
    />
  </view>
</template>

<script>
import TimeBlock from "./TimeBlock.vue";
export default {
  components: {
    TimeBlock,
  },
  emits: ["update"],
  props: {
    divide: {
      type: Number,
    },
    colNumber: {
      type: Number,
    },
    colTime: {
      type: Number,
    },
    unit: {
      type: String,
    },
  },
  data() {
    return {};
  },
  computed: {
    step() {
      // currently only support hour
      const unit = this.unit;
      const divide = this.divide;
      if (unit === "hour" || unit === "minute") {
        return 60 / divide;
      }
      if (unit === "day") {
        return 24 / divide;
      }
      return 60 / divide;
    },
    displayTime() {
      return this.padZero(this.colTime) + ": 00";
    },
    dividedTime() {
      const { divide, unit, colTime } = this;
      if (unit === "hour") {
        const divided = new Array(divide).fill().map((it, index) => {
          return `${this.padZero(colTime)}:${this.padZero(this.step * index)}:00`;
        });
        return divided;
      }
    },
    columnInfo() {
      return this.dividedTime.map((it, index) => {
        return {
          colIndex: index + 1,
          timeSpan: [it, this.stepAdd(it, this.step)],
          sn: (this.colNumber - 1) * this.divide + index + 1,
        }
      })
    }
  },
  created() {
    this.$emit('colInfo', this.columnInfo)
  },
  methods: {
    handleClick(position, sn) {
      this.$emit('blockClick', { position, sn });
    },
    padZero(value) {
      return `00${value}`.slice(-2);
    },
    stepAdd(input, step) {
      if (this.unit === "hour") {
        let [hour, minute] = input.split(":");
        hour *= 1;
        minute *= 1;
        minute += step;
        if (minute === 60) {
          return `${this.padZero(hour + 1)}:00:00`;
        }
        return `${this.padZero(hour)}:${this.padZero(minute)}:00`;
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.time-column {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.plain {
  width: 48px;
  height: 25px;
  text-align: center;
  line-height: 25px;
}
</style>
