<template>
  <div class="option-collector" :class="{ disabled }" @click="toggle()">
    <div class="option-collector-face">
      <slot name="face">
        <div v-if="face === 'single'" class="select-one">
          <span class="text">{{
            selected === null ? "Select One" : displaySelectedValue(selected)
          }}</span>
        </div>
        <div v-else-if="face === 'multiple'">
          <span class="total">{{ selectedOptionsArray.length }} Selected</span>
        </div>
        <template v-else-if="face === 'multiple-chips'">
          <div class="selected-chips">
            <div
              v-for="selectedValue in sortedSelectedOptions"
              :key="selectedValue"
              class="selected-chip"
              :class="classFromSelectedValue(selectedValue)"
              @click.stop
            >
              <span>{{ displaySelectedValue(selectedValue) }}</span>
              <font-awesome-icon
                icon="times"
                @click="removeChip(selectedValue)"
              />
            </div>
          </div>
        </template>
      </slot>
    </div>

    <div class="line" />
    <font-awesome-icon icon="caret-down" />

    <div
      ref="dropdownMenu"
      class="option-collector-menu"
      :style="dropdownStyles"
      :class="dropdownClasses"
      @click.stop
    >
      <div v-show="!hideSearch" class="option-collector-menu-top">
        <input
          v-model="searchText"
          type="text"
          class="form-control"
          placeholder="Search..."
          autocomplete="off"
          @input="searchTextChanged"
        />
        <button
          v-if="multiple"
          type="button"
          class="btn btn-default btn-sm"
          @click="toggleOptions"
        >
          <b>Select/Deselect All</b>
        </button>
      </div>
      <div
        v-if="optionHeader"
        class="option-collector-menu-bottom"
        :class="{ multiple }"
      >
        <div v-for="header in headers" :key="header[1]" class="option-group">
          <h6 :class="classFromHeaderPartitionGroup(header[0])">
            {{ header[1] }}
          </h6>
          <ol>
            <li
              v-for="option in optionsForHeader(header[0])"
              :key="option[optionKey]"
            >
              <label
                class="option-collector-label"
                @click="clickIfSelectOne(option)"
              >
                <input
                  v-if="multiple"
                  v-model="cselected"
                  type="checkbox"
                  :value="option[optionKey]"
                  @change="
                    toggleMultipleSelect(
                      option[optionKey],
                      $event.target.checked
                    )
                  "
                />
                <span style="padding-right: 1em">{{
                  option[optionValue]
                }}</span>
              </label>
            </li>
          </ol>
        </div>
      </div>
      <div v-else class="option-collector-menu-bottom" :class="{ multiple }">
        <ol>
          <li v-for="option in searchedOptions" :key="option[optionKey]">
            <label
              class="option-collector-label"
              @click="clickIfSelectOne(option)"
            >
              <input
                v-if="multiple"
                v-model="cselected"
                type="checkbox"
                :value="option[optionKey]"
                @change="
                  toggleMultipleSelect(option[optionKey], $event.target.checked)
                "
              />
              <span style="padding-right: 1em">{{ option[optionValue] }}</span>
            </label>
          </li>
        </ol>
      </div>
    </div>
  </div>
</template>

<script>
import { library } from "@fortawesome/fontawesome-svg-core";
import { faCaretDown, faTimes } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";

library.add(faCaretDown);
library.add(faTimes);

export default {
  components: {
    FontAwesomeIcon,
  },
  props: {
    face: {
      type: String,
      required: false,
      default: "single", // single, multiple, multiple-chips
    },
    countSelected: {
      type: Boolean,
      required: false,
      default: false,
    },
    options: {
      type: Array, // of objects
      required: true,
    },
    optionKey: {
      type: String, // property on options object that is unique
      required: false,
      default: "id",
    },
    selected: {
      /**
       * Type is either array of "value" or single "value" based on if multiple prop is true/false.
       * The "value" will be object if prop select-object is used; otherwise, the optionKey prop is used to get the value from the option object
       *
       */
      required: true,
    },
    selectObject: {
      type: Boolean,
      required: false,
      default: false,
    },
    selectedClasses: {
      type: Object, // Apply classes to selected items on default face using class name as key, bool|function as value ex {error: true}
      required: false,
      default: () => ({}),
    },
    headerClasses: {
      type: Object, // Apply classes to headers using class name as key, bool|function as value ex {error: true},
      required: false,
      default: () => ({}),
    },
    optionHeader: {
      type: Function, // passed option as argument
      required: false,
      default: null,
    },
    optionHeaderPartitionBy: {
      type: String, // the property on the option object to use for grouping under header
      required: false, // required if optionHeader supplied
      default: null,
    },
    optionValue: {
      type: String, // the property on the option object to output next to the checkmark
      required: false,
      default: "name",
    },
    defaultDirection: {
      type: String,
      required: false,
      default: "below",
    },
    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },
    hideSearch: {
      type: Boolean,
      required: false,
      default: false,
    },
    sort: {
      type: Boolean,
      required: false,
      default: true,
    },
  },

  data() {
    return {
      searchText: "",
      dropdownStyles: "",
      open: false,
    };
  },
  computed: {
    cselected: {
      get() {
        return this.selectedKeys;
      },
      set() {
        // see event instead
      },
    },
    multiple() {
      return this.face.startsWith("multiple");
    },
    selectedOptionsArray() {
      let selected = this.selected;
      if (!this.multiple) {
        selected = [selected];
      }

      // Filter out any invalid values
      selected = selected
        .map((selectedValue) => {
          return this.getOptionFromSelectedValue(selectedValue);
        })
        .filter((o) => o);

      return selected;
    },
    selectedKeys() {
      return this.selectedOptionsArray.map((o) => o[this.optionKey]);
    },
    sortedSelectedOptions() {
      return this.sortedOptions
        .filter((o) => this.selectedKeys.includes(o.id))
        .map((o) => o.id);
    },
    sortedOptions() {
      if (!this.sort) {
        return this.options;
      }

      return this.sortOptions(this.options);
    },
    headers() {
      if (this.optionHeader === null) {
        return [];
      }

      let groups = new Set(
        this.searchedOptions.map(
          (option) => option[this.optionHeaderPartitionBy]
        )
      );
      let output = Array.from(groups).map((group) => [
        group,
        this.optionHeader(group),
      ]);

      output.sort((a, b) => {
        a = a[1];
        b = b[1];

        if (a === b) {
          return 0;
        }

        return a > b ? 1 : -1;
      });

      return output;
    },
    searchedOptions() {
      return this.sortedOptions.filter((o) => {
        let value = String(o[this.optionValue]).toLowerCase();
        let search = this.searchText.toLowerCase();

        return value.indexOf(search) >= 0;
      });
    },
    direction() {
      // Re-cache computed prop if this value changes
      this.open;

      if (!this.$el) {
        return "below";
      }

      let boundingRect = this.$el.getBoundingClientRect();
      let dropdownHeight = this.$refs.dropdownMenu.offsetHeight;
      let viewportHeight = window.innerHeight;

      let availableBelow =
        boundingRect.bottom + dropdownHeight < viewportHeight;
      let availableAbove = boundingRect.top - dropdownHeight > 0;

      if (availableBelow && this.defaultDirection === "below") {
        return "below";
      } else if (availableAbove && this.defaultDirection === "above") {
        return "above";
      } else if (availableAbove) {
        return "above";
      } else {
        return this.defaultDirection;
      }
    },

    dropdownClasses() {
      let classes = [this.direction];
      if (this.open) {
        classes.push("open");
      }

      return classes;
    },
  },

  mounted() {
    window.addEventListener("click", this.checkOutsideClick);
    this.searchTextChanged(); // Recompute position
  },

  unmounted() {
    window.removeEventListener("click", this.checkOutsideClick);
  },

  methods: {
    searchTextChanged() {
      this.$nextTick(() => {
        let style = "";

        if (this.open) {
          style += "position: absolute; left: 0;";
        } else {
          style += "position: fixed; left: -1000px;";
        }

        if (this.direction === "above") {
          style += `top: calc(-${this.$refs.dropdownMenu.offsetHeight}px - 6px);`;
          style += "box-shadow: 0 -6px 12px rgba(0, 0, 0, 0.175);";
        } else {
          style += "top: 100%;";
          style += "box-shadow: 0 6px 12px rgba(0, 0, 0, 0.175);";
        }

        this.dropdownStyles = style;
      });
    },
    clickIfSelectOne(option) {
      if (!this.multiple) {
        let value = this.selectObject ? option : option[this.optionKey];

        this.$emit("update:selected", value);
        this.toggle();
      }
    },
    checkOutsideClick(event) {
      if (this.open && !this.$el.contains(event.target)) {
        this.toggle();
      }
    },
    sortOptions(options) {
      return Array.from(options).sort(function (a, b) {
        var nameA = a.name.toUpperCase();
        var nameB = b.name.toUpperCase();
        if (nameA < nameB) {
          return -1;
        }
        if (nameA > nameB) {
          return 1;
        }

        return 0;
      });
    },
    toggle() {
      if (this.disabled) {
        return;
      }
      this.open = !this.open;
      this.searchText = "";
      this.searchTextChanged();

      this.$nextTick(() => {
        if (this.open) {
          this.$el
            .querySelector(".form-control")
            .focus({ preventScroll: true });
        }
      });
    },
    toggleOptions() {
      let selected = this.selectedKeys;
      let searched = this.searchedOptions.map((o) => o[this.optionKey]);
      let exists = selected.filter((value) => searched.includes(value));

      if (exists.length === searched.length) {
        selected = selected.filter((value) => !searched.includes(value));
      } else {
        selected = Array.from(new Set(selected.concat(searched)));
      }

      if (this.selectedKeys.length === selected.length) {
        this.$emit("update:selected", []);
      } else {
        this.$emit("update:selected", selected);
      }
    },
    optionsForHeader(headerValue) {
      return this.searchedOptions.filter(
        (option) => option[this.optionHeaderPartitionBy] === headerValue
      );
    },
    optionFromKey(key) {
      let option = this.options.find(
        (option) => option[this.optionKey] === key
      );
      if (!option) {
        return null;
      }

      return option;
    },
    classFromHeaderPartitionGroup(groupId) {
      let classes = ["option-header"];

      for (let k in this.headerClasses) {
        if (typeof this.headerClasses[k] === "function") {
          if (this.headerClasses[k](groupId)) {
            classes.push(k);
          }
        } else {
          if (this.headerClasses[k]) {
            classes.push(k);
          }
        }
      }

      return classes;
    },
    classFromSelectedValue(selectedValue) {
      let option = this.getOptionFromSelectedValue(selectedValue);
      if (!option) {
        return {};
      }

      let classes = [];
      for (let k in this.selectedClasses) {
        if (typeof this.selectedClasses[k] === "function") {
          if (this.selectedClasses[k](option)) {
            classes.push(k);
          }
        } else {
          if (this.selectedClasses[k]) {
            classes.push(k);
          }
        }
      }

      return classes;
    },
    displaySelectedValue(selectedValue) {
      let option = this.getOptionFromSelectedValue(selectedValue);

      if (!option) {
        return "";
      }

      return option[this.optionValue];
    },
    getOptionFromSelectedValue(selectedValue) {
      let option;

      if (this.selectObject) {
        option = this.options.find(
          (o) => o[this.optionKey] === selectedValue[this.optionKey]
        );
      } else {
        option = this.optionFromKey(selectedValue);
      }

      return option;
    },
    toggleMultipleSelect(optionKeyValue, isSelected) {
      let selected = this.selectedOptionsArray;
      let option = this.optionFromKey(optionKeyValue);

      if (isSelected) {
        selected = selected.concat(option);
      } else {
        selected = selected.filter((o) => o[this.optionKey] !== optionKeyValue);
      }

      selected = this.selectObject
        ? selected
        : selected.map((o) => o[this.optionKey]);

      this.$emit("update:selected", selected);
    },
    removeChip(selectedValue) {
      if (this.disabled) {
        return;
      }

      let option = this.getOptionFromSelectedValue(selectedValue);
      if (option) {
        this.toggleMultipleSelect(option[this.optionKey], false);
      }
    },
  },
};
</script>

<style lang="scss" scope>
.option-collector {
  display: inline-flex;
  position: relative;
  align-items: center;
  border: 1px solid #e8e8e8;
  border-radius: 5px;
  cursor: pointer;
  min-height: 36px;

  .option-collector-face {
    padding: 0 0.5em;
    white-space: nowrap;
  }

  .line {
    border-left: 1px solid #e8e8e8;
    align-self: stretch;
    border-radius: 0 3px 3px 0;
  }

  .fa-caret-down {
    cursor: pointer;
    padding: 0 0.9em;
    z-index: 2;
  }

  &.disabled .fa-caret-down {
    color: #f3f3f3;
  }

  .select-one {
    width: 100%;
    justify-content: space-between;
    padding: 0 0.45em;
  }

  .selected-chips {
    display: flex;
    flex-wrap: wrap;

    max-height: calc(
      2em * 4.5
    ); // 4 lines max, show half a line to indicate need to scroll
    width: 100%;
    overflow: auto;
    overscroll-behavior: none;

    .selected-chip {
      display: flex;
      align-items: center;
      cursor: default;
      background-color: #ddd;
      margin: 0.25em;
      padding: 0.25em 0.5em;
      border-radius: 5px;
      font-size: 0.8em;

      .fa-times {
        cursor: pointer;
        padding: 0.25em;
        font-size: 0.85em;
        margin-left: 0.35em;

        &:hover {
          background-color: #999;
          border-radius: 5px;
        }
      }
    }
  }

  &.disabled .selected-chips .selected-chip {
    background-color: #f3f3f3;
    color: #a9a9a9;
    .fa-times {
      &:hover {
        background-color: transparent;
      }
    }
  }

  button {
    width: 100%;
  }

  .option-collector-menu {
    display: flex;

    &.above {
      flex-direction: column-reverse;
    }

    &.below {
      flex-direction: column;
    }

    &.open {
      margin: 2px 0;
      background-color: white;
      border: 1px solid rgba(0, 0, 0, 0.15);
      border-radius: 4px;
      z-index: 1000;
    }

    .option-collector-menu-top {
      padding: 0 0.5em 0.5em 0.5em;
      margin: 0.4em 0;

      .form-control {
        margin-bottom: 0.5em;
      }
    }

    .option-collector-menu-bottom {
      max-height: 400px;
      overflow: auto;
      overflow-x: -moz-hidden-unscrollable;
      overscroll-behavior: none;
      padding: 0 0.5em;

      &:not(.multiple) li > label {
        padding: 0.5em;
        &:hover {
          background-color: #eee;
        }
      }

      h6 {
        white-space: nowrap;
        font-weight: bold;
        border-bottom: 1px solid #eee;
        padding: 0.35em 0;

        &:first-child {
          margin-top: 0;
        }
      }

      ol {
        margin: 0;
        padding: 0;
        list-style-type: none;

        li {
          & > label {
            display: flex;
            align-items: center;
            padding: 0.35em;
            padding-right: 0;
            font-weight: normal;
            cursor: pointer;
          }

          white-space: nowrap;

          input[type="checkbox"] {
            margin: 0;
            margin-right: 0.2em;
            -moz-transform: scale(0.75);
          }
        }
      }
    }
  }
}
</style>
