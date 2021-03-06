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
    min-width: 100%;
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
  <div class="smart-select" :class="{ inline, multiple }">
    <span v-for="item in selecteds" :key="item[label.value]">
      <i @click="remove(item)"></i>{{ item[label.label] }}
    </span>
    <input type="text" ref="hint"
          v-model="hint"
          :tabindex="tabindex"
           @focus="editing = true"
           @blur="blur"
           @keydown.delete.stop="removeHint"
           @keydown.esc.prevent.stop="blur"
           @keydown.up.prevent.stop="hintUp"
           @keydown.down.prevent.stop="hintDown"
           @keydown.enter.prevent.stop="select(filterItems[index], $event)" />
    <ul v-show="editing" ref="list">
      <li ref="items"
          v-for="(item, i) in filterItems"
          :key="i"
          :class="{active: i === index, selected: selecteds.includes(item)}"
          @mouseenter.stop.prevent="index = i"
          @mousedown.stop.prevent="select(item, $event)">
        {{ item[label.label] }}
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    name: 'smart-select',

    props: {
      closeOnSelect: Boolean,
      inline: { type: Boolean, default: true },
      items: { type: Array, require: true },
      label: { type: Object, default: ()  => ({ label: 'label', value: 'value' }) },
      multiple: Boolean,
      tabindex: { type: Number, default: 0 },
      value: null
    },

    computed: {
      filterItems() {
        let items = this.items.slice();

        if (this.hint) {
          const reg = new RegExp(this.hint, 'i');
          items = items.filter(item => reg.test(item[this.label.label]));
        }

        return items.sort((a, b) => String(a[this.label.label]).localeCompare(b[this.label.label]));
      }
    },

    watch: {
      value: {
        immediate: true,
        handler: 'updateSelecteds'
      },

      items: 'updateSelecteds',

      filterItems() {
        this.index = 0;
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
            this._emit(this.selecteds.length ? this.selecteds.map(i => i[this.label.value]) : null);
          }
        }
      },

      select(item, e) {
        if (this.multiple) {
          const index = this.selecteds.indexOf(item);
          if (index === -1) {
            this.selecteds.push(item);
          } else {
            this.selecteds.splice(index, 1);
          }

          this._emit(this.selecteds.map(i => i[this.label.value]));
          if (this.closeOnSelect && !e.ctrlKey) {
            this.blur();
          }
        } else {
          this.selecteds = [item];
          this._emit(item[this.label.value]);
          this.blur();
        }
      },

      remove(item) {
        this.selecteds.splice(this.selecteds.indexOf(item), 1);
        this._emit(this.selecteds.length ? this.selecteds.map(i => i[this.label.value]) : null);
      },

      blur() {
        this.hint = '';
        this.editing = false;
        this.$refs.hint.blur();
      },

      updateSelecteds() {
        let values = this.value !== null ? [].concat(this.value) : [];
        this.selecteds = this.items.filter(item => {
          if (values.includes(item[this.label.value])) {
            values = values.filter(v => v !== item[this.label.value]);
            return true;
          }

          return false;
        });
      },

      _emit(result) {
        this.$emit('input', result);
        this.$emit('change', result);
      }
    },

    data() {
      return {
        hint: '',
        editing: false,
        index: 0,
        selecteds: []
      };
    }
  };
</script>
