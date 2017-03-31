<style>
  .smart-select {
    flex-grow: 1;
  }

  .smart-select.inline {
    display: flex;
    align-items: center;
  }

  .smart-select span {
    display: flex;
    float: left;
    padding: 0 7px;
    height: 30px;
    border-radius: 5px;
    user-select: none;
    white-space: nowrap;
    align-items: center;
  }

  .smart-select span i::before {
    font-style: normal;
    content: '✖';
    margin-right: 5px;
    cursor: pointer;
  }

  .smart-select ul {
    width: 100%;
    margin: 0;
    padding: 0;
    left: 0;
    top: 100%;
    position: absolute;
    background-color: #fff;
    list-style-type: none;
    max-height: 24rem;
    overflow-y: auto;
    z-index: 1;
  }

  .smart-select ul li {
    padding-left: 5px;
    cursor: pointer;
  }

  .smart-select ul li.active,
  .smart-select ul li.selected.active {
    background-color: #65605e;
    color: #fff;
  }

  .smart-select ul li.selected {
    background-color: #c4c4c4;
  }
</style>

<template>
  <div class="smart-select" :class="{inline: inline}">
    <span v-for="item in selecteds">
      <i @click="remove(item)"></i>{{ item[label.label] }}
    </span>
    <input type="text" ref="hint" :tabindex="tabindex"
           @focus="editing = true"
           @blur="blur"
           @keyup.delete="removeHint"
           @keyup.esc.prevent="blur"
           @keyup.up.prevent="hintUp"
           @keyup.down.prevent="hintDown"
           @keyup.enter.prevent="select(filterItems[index])"
           v-model="hint"/>
    <ul v-show="editing" ref="list">
      <li ref="items"
          :class="{active: i === index, selected: selecteds.includes(item)}"
          v-for="(item, i) in filterItems"
          @mouseenter="index = i"
          @mousedown="select(item)">
        {{ item[label.label] }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'smart-select',

  data() {
    return {
      hint: '',
      editing: false,
      index: 0,
      selecteds: []
    };
  },

  computed: {
    filterItems() {
      this.index = 0;
      if (this.hint) {
        const reg = new RegExp(this.hint, 'i');
        return this.items.filter(item => reg.test(item[this.label.label]));
      }
      return this.items;
    }
  },

  props: {
    items: {
      type: Array,
      require: true
    },

    value: null,

    multiple: {
      type: Boolean,
      default: false
    },

    label: {
      type: Object,
      default() {
        return {
          label: 'label',
          value: 'value'
        };
      }
    },

    inline: {
      type: Boolean,
      default: true
    },

    tabindex: {
      type: Number,
      default: 0
    }
  },

  methods: {
    resetScroll() {
      const itemRect = this.$refs.items[this.index].getBoundingClientRect();
      const listRect = this.$refs.list.getBoundingClientRect();
      if (itemRect.bottom > listRect.bottom) {
        this.$refs.list.scrollTop += itemRect.bottom - listRect.bottom;
      }
      if (itemRect.top < listRect.top) {
        this.$refs.list.scrollTop += itemRect.top - listRect.top;
      }
    },

    hintUp() {
      this.index = (this.index - 1 + this.filterItems.length) % this.filterItems.length;
      this.resetScroll();
    },

    hintDown() {
      this.index = (this.index + 1) % this.filterItems.length;
      this.resetScroll();
    },

    removeHint() {
      if (!this.hint.length) {
        const item = this.selecteds.pop();
        if (item) {
          this.hint = `${item[this.label.label]} `;
          this.$emit('change', this.selecteds.length ? this.selecteds.map(i => i[this.label.value]) : null);
        }
      }
    },

    select(item) {
      if (this.multiple) {
        if (this.selecteds.indexOf(item) === -1) {
          this.selecteds.push(item);
          this.hint = '';
          this.preventBlurOnce = true;
          this.$emit('change', this.selecteds.map(i => i[this.label.value]));
        }
      } else {
        this.selecteds = [item];
        this.$emit('change', item[this.label.value]);
        this.blur();
      }
    },

    remove(item) {
      this.selecteds.splice(this.selecteds.indexOf(item), 1);
      this.$emit('change', this.selecteds.length ? this.selecteds.map(i => i[this.label.value]) : null);
    },

    blur() {
      if (this.preventBlurOnce) {
        this.preventBlurOnce = false;
        this.$refs.hint.focus();
        return;
      }
      this.hint = '';
      this.editing = false;
      this.$refs.hint.blur();
    }
  },

  watch: {
    value: {
      immediate: true,
      handler() {
        if (this.value !== null && typeof this.value !== 'undefined') {
          const values = [].concat(this.value);
          this.selecteds = this.items.filter(item => values.includes(item[this.label.value]));
        }
      }
    }
  }
};
</script>
