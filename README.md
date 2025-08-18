# GridPicker (Time Blocks) uni-app

## Time select by range
![image](https://github.com/user-attachments/assets/1314921e-f664-4d06-804e-655b115bdce9)

## Time pick in blocks
![image](https://github.com/user-attachments/assets/4bf10673-b3c8-4683-b9b8-6fce2cae6fb0)

### props supported
```
timeRange: {
  type: Array,
  default() {
    // example(when unit === 'hour')
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
    // return (when unit === 'hour'): ["08:00:00", "20:00:00"];
    return [];
  },
},
```


