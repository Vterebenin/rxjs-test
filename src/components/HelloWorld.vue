<template>
    <div >
        <div>{{ count }} {{ someOtherShit }}</div>

        <!-- simple usage -->
        <button v-stream:click="plus$">Add on Click</button>

        <button v-stream:click="{ subject: plus$, data: minusDelta1, options:{once:true} }">Add on Click (Option once:true)</button>

        <!-- you can also stream to the same subject with different events/data -->
        <button
                v-stream:click="{ subject: minus$, data: minusDelta1 }"
                v-stream:mousemove="{ subject: minus$, data: minusDelta2 }">
            Minus on Click &amp; Mousemove
        </button>

        <pre>{{ $data }}</pre>

        <my-button v-stream:click="plus$"></my-button>
    </div>
</template>

<script>
  import { merge } from 'rxjs'
  import { map, pluck, startWith, scan } from 'rxjs/operators'

  export default {
    name: 'HelloWorld',
    data () {
      return {
        minusDelta1: -1,
        minusDelta2: -1
      }
    },
    components: {
      myButton: {
        template: `<button @click="$emit('click')">MyButton</button>`
      }
    },
    domStreams: ['plus$', 'minus$'],
    subscriptions() {

      return {
        count: merge(
          this.plus$.pipe(map(() => 1)),
          this.minus$.pipe(pluck('data'))
        ).pipe(
          startWith(0),
          scan((total, change) => total + change)
        ),
        someOtherShit: merge(
          this.plus$.pipe(map(() => 1)),
          this.minus$.pipe(pluck('data'))
        ).pipe(
          startWith(0),
          scan((total, change) => total + change)
        ),
      }
    },
    created () {
      //Speed up mousemove minus delta after 5s
      setTimeout(() => {
        this.minusDelta2 = -5
      }, 5000)
    },
    props: {
      msg: String
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
    h3 {
        margin: 40px 0 0;
    }
    ul {
        list-style-type: none;
        padding: 0;
    }
    li {
        display: inline-block;
        margin: 0 10px;
    }
    a {
        color: #42b983;
    }
</style>
