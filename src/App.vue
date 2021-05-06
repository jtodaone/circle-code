<template>
  <div id="app">
    <input type="text" v-model="name">
    <span @click="createSymbol(name)">new</span>
    <figure class="canvas" ref="canvas"></figure>
  </div>
</template>

<script>
import svg5 from 'svg5'
import md5 from 'crypto-js/md5'

export default {
  name: 'App',
  data() {
    return {
      name: '',
      lineWidth: 10
    }
  },
  mounted() {
    svg5.createSVG(1000, 1000)
  },
  methods: {
    createSymbol(name) {
      this.$refs.canvas.childNodes.forEach(e => {e.remove()})
      svg5.clear()
      svg5.noFill()
      svg5.stroke(0)
      svg5.strokeWeight(this.lineWidth)
      
      const firstHash = md5(name).toString()
      let secondHash = []

      for (let i = 0; i < 8; i++) {
        secondHash.push(firstHash.slice(i * 4, (i + 1) * 4))
      }

      secondHash = secondHash.map(md5)

      console.log(secondHash)

      secondHash.forEach((hash, hashIndex) => {
        let bytes = hash.words.map(word => {
          const binary = (word>>>0).toString(2)
          return '0'.repeat((32 - binary.length)) + binary
        })
        bytes = bytes.reduce((prev, curr) => { return prev + curr })
        bytes = bytes.split('').reduce((prev, curr) => {
          if (prev.length == 0) {
            return [{
              draw: curr == 1,
              length: 1
            }]
          } else if (prev[prev.length - 1].draw && curr == 1) {
            prev[prev.length - 1].length += 1
            return prev
          } else if (!prev[prev.length - 1].draw && curr == 0) {
            prev[prev.length - 1].length += 1
            return prev
          } else {
            return [...prev, {
              draw: curr == 1,
              length: 1
            }]
          }
        }, [])

        let linePosition = 0
        const max = 128 /*- hashIndex * 5*/
        const step = 360 / max

        bytes.slice(0, max).forEach((line, lineIndex) => {
          if (line.draw) svg5.arc(svg5.width / 2, svg5.height / 2, svg5.width * 0.2 + (this.lineWidth * 3 * hashIndex), svg5.height * 0.2 + (this.lineWidth * 3 * hashIndex), linePosition * step, (linePosition + line.length) * step)
          linePosition += line.length
        })
      })

      svg5.render('.canvas')
    }
  }
}
</script>

<style>
</style>
