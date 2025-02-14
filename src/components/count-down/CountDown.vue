<template>
  <slot :remain="convertRemainTime(remain)">{{ formatRemainTime(remain, format) }}</slot>
</template>

<script lang="ts">
import { requestAnimFrame, cancelAnimFrame } from '@/utils/raf'
import { formatRemainTime, convertRemainTime } from '@/utils/format'
import { defineComponent } from 'vue'

interface Data {
  remain: number
  endingTime: number
  rafId: number
}

export default defineComponent({
  props: {
    ms: { type: Boolean, default: false },
    remainTime: { type: Number, default: 0 },
    format: { type: String, default: 'HH:mm:ss' }, // d天 HH:mm:ss.SSS
  },

  data: function (): Data {
    return {
      rafId: 0,
      remain: 0,
      endingTime: 0, // 结束时间 = 当前时间 + 剩余秒数
    }
  },

  watch: {
    remainTime: {
      immediate: true,
      handler: 'start',
    },
  },

  methods: {
    ...{ formatRemainTime, convertRemainTime },

    start(): void {
      if (!this.remainTime) {
        return
      }
      this.remain = this.remainTime
      this.endingTime = Date.now() + this.remain
      this.tick()
    },

    tick(): void {
      this.rafId = requestAnimFrame(() => {
        const remain = Math.max(this.endingTime - Date.now(), 0)

        if (remain === 0) {
          this.remain = 0
          cancelAnimFrame(this.rafId)
          this.$emit('finish')
          return
        }

        const isDiffSecond = Math.abs(remain - this.remain) > 986 // 17 * 58

        if (this.ms || isDiffSecond) {
          this.remain = remain
        }

        this.tick()
      })
    },
  },
})
</script>
