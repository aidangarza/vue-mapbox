<template>
  <div style="display: none">
    <!-- @slot Slot for popup content -->
    <slot/>
  </div>
</template>

<script>
import baseMixin from '../../lib/mixin'

/**
 * Popup component.
 * @see See [Mapbox Gl JS Popup](https://www.mapbox.com/mapbox-gl-js/api/#popup)
 */
export default {
  name: 'Popup',
  mixins: [baseMixin],
  props: {
    /**
     * If `true`, a close button will appear in the top right corner of the popup.
     * Mapbox GL popup option.
     */
    closeButton: {
      type: Boolean,
      default: true
    },
    /**
     * Mapbox GL popup option.
     * If `true`, the popup will closed when the map is clicked. .
     */
    closeOnClick: {
      type: Boolean,
      default: true
    },
    /**
     * Mapbox GL popup option.
     * A string indicating the popup's location relative to the coordinate set.
     * If unset the anchor will be dynamically set to ensure the popup falls within the map container with a preference for 'bottom' .
     *  'top', 'bottom', 'left', 'right', 'top-left', 'top-right', 'bottom-left', 'bottom-right'
     */
    anchor: {
      validator(value) {
        let allowedValues = ['top', 'bottom', 'left', 'right', 'top-left', 'top-right', 'bottom-left', 'bottom-right']
        return (typeof value === 'string' && allowedValues.indexOf(value) !== -1) || value === undefined
      },
      default: undefined
    },
    /**
     * Mapbox GL popup option.
     * A pixel offset applied to the popup's location
     * a single number specifying a distance from the popup's location
     * a PointLike specifying a constant offset
     * an object of Points specifing an offset for each anchor position Negative offsets indicate left and up.
     */
    offset: {
      type: [Number, Object, Array],
      default: () => [0, 0]
    },
    coordinates: {
      type: Array
    },

    /**
     * Component option.
     * If `true`, popup treats data in deafult slot as plain text
     */
    onlyText: {
      type: Boolean,
      default: false
    }
  },

  data() {
    return {
      initial: true,
      popup: undefined
    }
  },

  computed: {
    /**
     * true if popup is open
     * @returns {*}
     */
    isOpen() {
      if (this.popup !== undefined) {
        return this.popup.isOpen()
      }
      return false
    }
  },

  mounted() {
    this.$_checkMapTree()
  },

  beforeDestroy() {
    if (this.map) {
      this.$_emitMapEvent('removed', { popup: this.popup })
      this.popup.remove()
    }
  },

  watch: {
    coordinates(lngLat) {
      if (this.initial) return
      this.popup.setLngLat(lngLat)
    }
  },

  methods: {
    $_deferredMount(payload) {
      this.map = payload.map
      this.$_addPopup()
      this.initial = false
      payload.component.$off('load', this.$_deferredMount)
    },

    $_addPopup() {
      this.popup = new this.mapbox.Popup({ ...this._props })
      if (this.coordinates !== undefined) this.popup.setLngLat(this.coordinates)
      if (this.$slots.default !== undefined) {
        if (this.onlyText) {
          if (this.$slots.default[0].elm.nodeType === 3) {
            let tmpEl = document.createElement('span')
            tmpEl.appendChild(this.$slots.default[0].elm)
            this.popup.setText(tmpEl.innerText)
          } else {
            this.popup.setText(this.$slots.default[0].elm.innerText)
          }
        } else {
          this.popup.setDOMContent(this.$slots.default[0].elm)
        }
      }
      this.popup.addTo(this.map)
      this.$_emitMapEvent('added', { popup: this.popup })

      this.popup.on('close', this.$_onClose)
      this.popup.on('open', this.$_onOpen)

      if (this.$parent.marker !== undefined) {
        this.$parent.marker.setPopup(this.popup)
      } else {
        this.$parent.$once('added', ({ marker }) => {
          marker.setPopup(this.popup)
        })
      }
    },

    $_onClose() {
      /**
       * Popup close event
       * @event close
       * @type {object}
       */
      this.$_emitMapEvent('close', { popup: this.popup })
    },

    $_onOpen() {
      /**
       * Popup close event
       * @event open
       * @type {object}
       */
      this.$_emitMapEvent('open', { popup: this.popup })
    }
  }
}
</script>
