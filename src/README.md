```vue
<template>
  <div>
    <wh-list :data-src="createRequest()"/>
  </div>
</template>

<script>
import 'element-ui/lib/theme-chalk/index.css'
import WhList from '../src/WhList.vue'
import Axios from "axios";

export default {
  name: 'App',
  components: { WhList },
  data() {
    return {
    }
  },
  methods: {
    createRequest() {
      return Axios.create({
        baseURL: 'http://test',
        url: '/test'
      })
    }
  }
}
</script>

<style scoped>

</style>

```
