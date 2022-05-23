<template>
  <div class="box">อีหยังวะนิ</div>
</template>

<script>
import _ from 'lodash'

export default {
  props: {
    fields: {
      type: Array,
      default () {
        return []
      }
    },
    data: {
      type: Array,
      required: true
    },
    pageLength: {
      type: [Array, Number],
      default () {
        return [10, 25, 50]
      }
    },
    loading: {
      type: Boolean,
      default: false
    },
    lengthChange: {
      type: Boolean,
      default: true
    },
    filterable: {
      type: Boolean,
      default: true
    },
    pagination: {
      type: Boolean,
      default: true
    },
    tableClass: {
      type: String,
      default: 'table is-striped is-narrow is-fullwidth'
    },
    inputClass: {
      type: String,
      default: ''
    },
    scrollable: {
      type: Boolean,
      default: false
    },
    bodyHeight: {
      type: String,
      default: '300px'
    }
  },

  data () {
    return {
      perPage: 10,
      from: 0,
      tableFilter: '',
      columnsFilter: {},
      sort1: { field: '', order: '' },
      sort2: { field: '', order: '' },
      sort3: { field: '', order: '' },

      regexSearch: false
    }
  },

  computed: {
    columnslength () {
      return Object.keys(this.columnsFilter).length === 0
    },
    filteredData () {
      const vm = this
      return this.columnSearchable ? _.filter(this.data, (data) => {
        if (Object.keys(this.columnsFilter).length === 0) return true
        let state = true
        for (const i in vm.columnsFilter) {
          if (vm.columnsFilter[i].length === 0) continue
          state = state && String(vm.getObjectData(data, i)).toLowerCase().indexOf(vm.columnsFilter[i].toLowerCase()) >= 0
        }
        return state
      }) : _.filter(this.data, (data) => {
        for (const i in vm.fields) {
          if (this.regexSearch) {
            return String(vm.getObjectData(data, vm.fields[i].name)).search(vm.tableFilter) >= 0
          } else if (String(vm.getObjectData(data, vm.fields[i].name)).toLowerCase().indexOf(vm.tableFilter.toLowerCase()) >= 0) return true
        }
      })
    },
    dataSort () {
      return _.orderBy(this.filteredData, [
        this.sort1.field, this.sort2.field, this.sort3.field
      ], [this.sort1.order, this.sort2.order, this.sort3.order])
    },
    dataSet () {
      if (this.scrollable) return this.dataSort
      return this.dataSort.slice(this.from, this.from + this.perPage)
    },
    lastPage () {
      return Math.ceil(this.filteredData.length / this.perPage)
    },
    currentPage () {
      return Math.ceil(this.from / this.perPage) + 1
    },
    columnSearchable () {
      return this.fields.filter((field) => {
        return typeof field.search !== 'undefined' ? field.search : 0
      }).length > 0 ? 1 : 0
    },
    floatHead () {
      let className = 'cursorPointer'
      if (this.scrollable) className += ' floatHead'
      return className
    }
  },

  watch: {
    dataSet (data) {
      this.$emit('currentData', data)
    },
    pageLength (data) {
      this.perPage = Array.isArray(data) ? data[0] : data
    },
    tableFilter () {
      this.from = 0
    },
    columnsFilter: {
      handler () {
        this.from = 0
      },
      deep: true
    }
  },

  beforeMount () {
    if (this.scrollable) {
      this.perPage = 1000000
    } else {
      this.perPage = Array.isArray(this.pageLength) ? this.pageLength[0] : this.pageLength
    }
  },

  methods: {
    previousPage () {
      if (this.from - this.perPage >= 0) {
        this.from = this.from - this.perPage
      }
    },
    nextPage () {
      if (this.from + this.perPage < this.filteredData.length) {
        this.from = this.from + this.perPage
      }
    },
    toFirstPage () {
      this.from = 0
    },
    toLastPage () {
      this.from = Math.floor((this.filteredData.length - 1) / this.perPage) * this.perPage
    },
    sortField (field) {
      if (this.sort1.field === field.name) {
        this.sort1.order = this.sort1.order !== 'asc' ? 'asc' : 'desc'
      } else {
        Object.assign(this.sort3, this.sort2)
        Object.assign(this.sort2, this.sort1)
        this.sort1 = {
          field: field.name,
          order: 'asc'
        }
      }
    },
    getObjectData (data, field) {
      let o = data
      field.split('.').forEach(key => {
        o = o[key] ? o[key] : ''
      })
      return o
    }
  }
}
</script>

<style>
@import './fontawesome/css/fontawesome.min.css';
@import './fontawesome/css/fa-solid.min.css';

.datatable-wrapper {
  overflow: auto;
}

.datatable-wrapper table thead th,
.datatable-wrapper table thead td {
  white-space: nowrap;
}

.datatable-wrapper::-webkit-scrollbar-track {
  box-shadow: inset 0 0 3px rgba(0, 0, 0, 0.3);
  -webkit-box-shadow: inset 0 0 3px rgba(0, 0, 0, 0.3);
  border-radius: 10px;
  background-color: rgba(255, 255, 255, 0.1);
}

.datatable-wrapper::-webkit-scrollbar {
  width: 8px;
  height: 8px;
  background-color: rgba(255, 255, 255, 0.1);
}

.datatable-wrapper::-webkit-scrollbar-thumb {
  border-radius: 10px;
  box-shadow: inset 0 0 6px rgba(0, 0, 0, .3);
  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, .3);
  background-color: rgba(255, 255, 255, 0.1);
}

.floatHead {
  position: sticky;
  top: 0;
  box-shadow: 0 2px 2px -1px rgba(0, 0, 0, 0.1)
}
</style>
