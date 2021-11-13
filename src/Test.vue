<template>
  <div>
    <h1>Single</h1>
    <table>
      <thead>
        <th>Config</th>
        <th>Dropdown</th>
        <th>Value</th>
      </thead>
      <tbody>
        <tr>
          <td>Plain</td>
          <td>
            <option-collector :options="options" :selected.sync="selectedOne" />
          </td>
          <td>{{ selectedOne }}</td>
        </tr>
        <tr>
          <td>Headers</td>
          <td>
            <option-collector
              :options="options"
              :selected.sync="selectedOne"
              option-header-partition-by="tag"
              :option-header="createHeader"
            />
          </td>
          <td>{{ selectedOne }}</td>
        </tr>
        <tr>
          <td>Object</td>
          <td>
            <option-collector
              :options="options"
              :selected.sync="selectedOne"
              :select-object="true"
            />
          </td>
          <td>{{ selectedOne }}</td>
        </tr>
        <tr>
          <td>Disabled</td>
          <td>
            <option-collector
              :options="options"
              :selected.sync="selectedOne"
              disabled
            />
          </td>
          <td>{{ selectedOne }}</td>
        </tr>
      </tbody>
    </table>

    <h1>Multiple</h1>
    <table>
      <thead>
        <th>Config</th>
        <th>Dropdown</th>
        <th>Value</th>
      </thead>
      <tbody>
        <tr>
          <td>Plain</td>
          <td>
            <option-collector
              :options="options"
              :selected.sync="selectedMultiple"
              face="multiple"
            />
          </td>
          <td>{{ selectedMultiple }}</td>
        </tr>
        <tr>
          <td>Headers</td>
          <td>
            <option-collector
              :options="options"
              :selected.sync="selectedMultiple"
              option-header-partition-by="tag"
              :option-header="createHeader"
              face="multiple"
            />
          </td>
          <td>{{ selectedMultiple }}</td>
        </tr>
        <tr>
          <td>Object</td>
          <td>
            <option-collector
              :options="options"
              :selected.sync="selectedMultiple"
              :select-object="true"
              face="multiple"
            />
          </td>
          <td>{{ selectedMultiple }}</td>
        </tr>
      </tbody>
    </table>

    <h1>Multiple Chips</h1>
    <table>
      <thead>
        <th>Config</th>
        <th>Dropdown</th>
        <th>Value</th>
      </thead>
      <tbody>
        <tr>
          <td>Plain</td>
          <td>
            <option-collector
              :options="options"
              :selected.sync="selectedMultiple"
              face="multiple-chips"
            />
          </td>
          <td>{{ selectedMultiple }}</td>
        </tr>
        <tr>
          <td>Disabled</td>
          <td>
            <option-collector
              :options="options"
              :selected.sync="selectedMultiple"
              face="multiple-chips"
              disabled
            />
          </td>
          <td>{{ selectedMultiple }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import OptionCollector from "./OptionCollector.vue";

export default {
  components: {
    OptionCollector,
  },
  data() {
    return {
      selectedOne: null,
      selectedMultiple: [],
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
      ],
    };
  },
  methods: {
    createHeader(tag) {
      let count = this.options.filter((o) => o.tag === tag).length;

      return `${count} with tag "${tag}"`;
    },
  },
};
</script>

<style lang="scss">
table {
  width: 800px;
  border-collapse: collapse;
  border-radius: 5px;

  th {
    background-color: black;
    color: white;
    text-align: left;
    padding: 0.25em;
  }

  td {
    padding: 1em 0;
  }
}
.value {
  margin: 1em;
}
</style>
