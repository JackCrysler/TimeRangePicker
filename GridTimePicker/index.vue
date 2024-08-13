<template>
  <view class="grid-time-picker">
    <view><slot name="header"></slot></view>
    <view class="time-grid">
      <TimeColumn
        v-for="(col, idx) in columns"
        :divide="divide"
        :unit="unit"
        :col-time="col.time"
        :col-number="idx + 1"
        :key="'column' + idx"
        @block-click="updatePicker"
        @colInfo="collectColumns"
      ></TimeColumn>
    </view>
    <view><slot name="footer"></slot></view>
  </view>
</template>

<script>
import { computed } from 'vue';
import TimeColumn from "./TimeColumn.vue";
export default {
  components: {
    TimeColumn,
  },
  props: {
    timeRange: {
      type: Array,
      default() {
        return ["08:00:00", "20:00:00"];
      },
    },
    disabledRanges: {
      type: Array,
      default() {
        // disabled ranges: [{ from, to }, ...]
        return [];
      },
    },
    limit: {
      type: Number,
      default() {
        // selectable value range by unit
        return 2;
      },
    },
    unit: {
      type: String,
      default() {
        // unit ['week', 'day', 'minute', ...] to be supported!
        return "hour";
      },
    },
    divide: {
      // split unit into equal parts, ensure the part be an integer
      type: Number,
      default() {
        return 4;
      },
    },
    value: {
      type: Array,
      default() {
        return [];
      },
    },
  },
  data() {
    return {
      events: {},
      selected: [],
      SNs: {},
      current: null,
    };
  },
  provide() {
    return {
      eventBus: { listen: this.listen, broadcast: this.broadcast },
      disabledSNs: computed(() => this.disabledBlocks.flat()),
    };
  },
  computed: {
    columns() {
      const [from, to] = this.timeRange;
      if (this.unit === "hour") {
        const [hour, minute] = from.split(":");
        let [toHour, toMinute] = to.split(":");
        toMinute * 1 > 0 && (toHour += 1);
        const cols = new Array(toHour - hour + 1).fill().map((it, index) => ({
          time: hour * 1 + index,
        }));
        return cols;
      }
      return [];
    },
    disabledBlocks() {
      if (Array.isArray(this.disabledRanges) && this.disabledRanges.length) {
        const ret =  this.disabledRanges.map(({from, to}) => {
          return this.rangeToList(this.getBlocksFromRange([from, to]));
        })
        return ret
      }
      return []
    },
  },
  watch: {
    value: {
      handler(cur) {
        if (this.value.filter(it => it).length === 2) {
          const initBlocks = this.getBlocksFromRange(this.value);
          this.selected = initBlocks;
          this.current = initBlocks[Math.floor(initBlocks.length / 2)];
          const renderMsg = this.generateRender(initBlocks);
          this.broadcast("update", renderMsg);
        } else {
          this.reset()
        }
      },
      immediate: true
    },
  },
  methods: {
    listen(event, handler) {
      const handlerList = this.events[event] || [];
      handlerList.push(handler);
      this.events[event] = handlerList;
    },
    broadcast(event, ...args) {
      (this.events[event] || []).forEach((handler) => handler(...args));
    },
    collectColumns(cols) {
      cols.forEach((col) => {
        this.SNs = {
          ...this.SNs,
          [col.sn]: col.timeSpan,
        };
      });
    },
    onActiveBorder(sn) {
      const [start, end] = [...this.selected];
      if (start === end) {
        return [];
      }
      if (sn === start) {
        return [sn + 1, end];
      }
      if (sn === end) {
        return [start, end - 1];
      }
    },
    offActiveBorder(sn) {
      let blocks = [...this.selected];
      if (blocks.length === 2 && sn > blocks[0] && sn < blocks[1]) {
        const middle = (blocks[0] + blocks[1]) / 2; // between the selected blocks
        blocks[+(middle <= sn)] = sn;
      } else {
        blocks = blocks.concat([sn]).sort((a, b) => a - b); // outside the selected blocks
        blocks.length > 2 && blocks.splice(1, 1); // del surplus point
      }

      if (blocks.length === 1) {
        blocks[1] = blocks[0]; // if end is void, use start as end
      }
      return blocks;
    },
    selectableBorder(blocks) {
      if (blocks.length === 0) {
        return [];
      }
      const totalBlocks = this.columns.length * this.divide;

      const dValue = this.limit * this.divide;
      const start = blocks[0] || 0;
      const end = blocks[1] || totalBlocks;
      const left = end - dValue + 1;
      const right = start + dValue - 1;
      const border = [left < 0 ? 0 : left, right > totalBlocks ? totalBlocks : right];
      const ret = this.disabledBlocks.length ? this.breakBorder(border) : border;
      return ret;
    },
    breakBorder(selectable) {
      const disabledBks2D = [...this.disabledBlocks];
      const selectableBks = this.rangeToList(selectable);
      const currentSN = this.current;
      
      const f = (disabledBks) => {
        const dbksLen = disabledBks.length;
        if (
          dbksLen > 0 &&
          this.isOverlap(selectableBks, disabledBks) &&
          !isNaN(currentSN)
        ) {
          if (currentSN < disabledBks[0]) {
            return [selectable[0], disabledBks[0] - 1];
          }
          if (currentSN > (disabledBks[1] || disabledBks[0])) {
            return [disabledBks[dbksLen - 1] + 1, selectable[1]];
          }
        }
      }

      const fragments = disabledBks2D.map(it => f(it)).filter(it => it).map(it=> this.rangeToList(it));
      if (fragments.length) {
        const o = (arr1, arr2) => arr1.filter(item => arr2.includes(item));
        const overlaped = fragments.reduce((prev, next) => prev.length === 0 ? next : o(next, prev), [])
        const ret = [overlaped[0], overlaped.pop()];
        // console.log('overlaped range', ret)
        return ret
      }
      return selectable;
    },
    generateRender(blocks) {
      const borders = this.selectableBorder(blocks);
      const ret = {
        actived: this.rangeToList(blocks),
        border: this.rangeToList(borders),
      };
      return ret;
    },
    reset() {
      this.selected = [];
      this.current = null;
      const renderMsg = this.generateRender([]);
      this.broadcast("update", renderMsg);
    },
    updatePicker(info) {
      const { sn } = info;
      let blocks = [...this.selected];

      if (sn === blocks[0] || sn === blocks[1]) {
        blocks = this.onActiveBorder(sn); // located on the active border
      } else {
        blocks = this.offActiveBorder(sn); // located off the active border
      }
      this.selected = blocks;
      this.current = sn;

      const renderMsg = this.generateRender(blocks);
      this.broadcast("update", renderMsg);

      const pickerValue = blocks.length === 2 ? [this.SNs[blocks[0]][0], this.SNs[blocks[1]][1]] : [];
      this.$emit("change", pickerValue);
    },
    rangeToList(range, cb) {
      if (Array.isArray(range) && range.length === 2) {
        const [from, to] = range;
        return new Array(to - from + 1).fill().map((it, index) => {
          return cb ? cb(from + index) : from + index;
        });
      }
      return range;
    },
    getBlocksFromRange(range) {
      if (range.length === 0) return range;
      const [s] = this.timeRange;
      const [r, er] = range;

      const [h, m] = this.timeDiff(s, r);
      const [eh, em] = this.timeDiff(s, er);

      const fromSN = this.stepsDiff(h, m) + 1;
      const toSN = this.stepsDiff(eh, em);
      return [fromSN, toSN];
    },
    stepsDiff(hour, minute) {
      return hour * this.divide + minute / (60 / this.divide);
    },
    timeDiff(from, to) {
      const [hour, minute, second] = from.split(":");
      const [toHour, toMinute, toSecond] = to.split(":");
      const diffSeconds =
        this.toSeconds(toHour, toMinute, toSecond) - this.toSeconds(hour, minute, second);
      return this.fromSeconds(diffSeconds);
    },
    toSeconds(hours, minutes, seconds) {
      return hours * 3600 + minutes * 60 + seconds * 1;
    },
    fromSeconds(totalSeconds) {
      let hours = Math.floor(totalSeconds / 3600);
      let minutes = Math.floor((totalSeconds % 3600) / 60);
      let seconds = totalSeconds % 60;
      return [hours, minutes, seconds];
    },
    isOverlap(arr1, arr2) {
      return arr1.some((item) => arr2.includes(item));
    },
  },
};
</script>

<style lang="scss" scoped>
.grid-time-picker {
  flex: 1;
  display: flex;
  flex-direction: column;
  width: 0;
}
.time-grid {
  position: relative;
  flex: 1;
  display: flex;
  flex-direction: row;
  gap: 8px;

  overflow: auto;
  .cover {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    z-index: 1;
  }
}
</style>
