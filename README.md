# Option Collector

A vue dropdown component that handles objects in a convenient way.

## Install

```
npm install --save option-collector
```

## Usage

```vue
<template>
  <dropdown :options="options" :selected.sync="selected">

  <p>Selected Object has optionKey (id by default) value: {{ selected }}</p>
</template>

<script>
import dropdown from 'option-collector';

export default {
  components: {
    dropdown
  },
  data() {
    return {
      selected: null,
      options: [
        {
          id: 1,
          name: "A",
          tag: "some tag",
        },
        {
          id: 2,
          name: "C",
          tag: "some tag",
        },
        {
          id: 3,
          name: "B",
          tag: "other tag",
        },
      ]
    }
  }
}
</script>
```

## Options

Look at src/Test.vue or the props on OptionCollector.vue

### Dev Testing
```
npm run serve
```

### Compiles and minifies for npm
```
npm run build
```

### Lints and fixes files
```
npm run lint
```
