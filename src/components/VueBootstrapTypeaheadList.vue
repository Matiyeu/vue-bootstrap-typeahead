<template>
  <div class="list-group shadow">
    <vue-bootstrap-typeahead-list-item
        v-for="(item, id) in matchedItems" :key="id"
        :data="item.data"
        :html-text="highlight(item.text)"
        :background-variant="backgroundVariant"
        :text-variant="textVariant"
        @click.native="handleHit(item, $event)"
    >
      <template v-if="$scopedSlots.suggestion" slot="suggestion" slot-scope="{ data, htmlText }">
        <slot name="suggestion" v-bind="{ data, htmlText }" />
      </template>
    </vue-bootstrap-typeahead-list-item>
  </div>
</template>

<script>
import VueBootstrapTypeaheadListItem from './VueBootstrapTypeaheadListItem.vue'
import accentize from 'accentize'
import removeAccents from 'remove-accents'

function sanitize(text) {
  return text.replace(/</g, '&lt;').replace(/>/g, '&gt;')
}

function escapeRegExp(str) {
  return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
}

export default {
  name: 'VueBootstrapTypeaheadList',

  components: {
    VueBootstrapTypeaheadListItem
  },

  props: {
    data: {
      type: Array,
      required: true,
      validator: d => d instanceof Array
    },
    query: {
      type: String,
      default: ''
    },
    backgroundVariant: {
      type: String
    },
    textVariant: {
      type: String
    },
    maxMatches: {
      type: Number,
      default: 10
    },
    minMatchingChars: {
      type: Number,
      default: 2
    },
    rawResults: {
      type: Boolean
    },
    ignoreAccents: {
      type: Boolean,
      default: false
    }
  },

  computed: {
    highlight() {
      return (text) => {
        text = sanitize(text)
        if (this.query.length === 0) {
          return text
        }
        const re = this.ignoreAccents ? accentize(this.escapedQuery) : new RegExp(this.escapedQuery, 'gi')

        return text.replace(re, `<strong>$&</strong>`)
      }
    },

    escapedQuery() {
      const r = escapeRegExp(sanitize(this.query))
      return this.ignoreAccents ? removeAccents(r) : r
    },

    matchedItems() {
      if (this.rawResults) {
        return this.data
      }

      if (!this.query || this.query.length === 0 || this.query.length < this.minMatchingChars) {
        return []
      }

      const re = this.ignoreAccents ? accentize(this.escapedQuery) : new RegExp(this.escapedQuery, 'gi')

      // Filter, sort, and concat
      return this.data
        .filter(i => i.text.match(re) !== null)
        .sort((a, b) => {
          const aIndex = a.text.indexOf(a.text.match(re)[0])
          const bIndex = b.text.indexOf(b.text.match(re)[0])

          if (aIndex < bIndex) { return -1 }
          if (aIndex > bIndex) { return 1 }
          return 0
        }).slice(0, this.maxMatches)
    }
  },

  methods: {
    handleHit(item, evt) {
      this.$emit('hit', item)
      evt.preventDefault()
    }
  }
}
</script>