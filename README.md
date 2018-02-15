# vue-h-select
[![npm version](https://badge.fury.io/js/vue-h-select.svg)](https://badge.fury.io/js/vue-h-select)
[![npm](https://img.shields.io/npm/l/express.svg)](http://opensource.org/licenses/MIT)
[![Build Status](https://travis-ci.org/zont/vue-h-select.svg?branch=master)](https://travis-ci.org/zont/vue-h-select)

> Edge 12+, FF 36+, Chrome 49+, or use translation from ES2105 to ES5

## Introduction

`Under construction`

## Setup
```bash
npm install vue-h-select
```

## Example
```javascript
import smartSelect from 'vue-h-select';

export default {
  template: `
    <div>
      <smart-select :items="options" v-model="value" multiple>
    </div>
  `,

  components: {
    smartSelect
  },

  data() {
    return {
      value: 2,
      options: [
        {label: 'Opt 1', value: 1},
        {label: 'Opt 2', value: 2}
      ]
    };
  }
};
```
