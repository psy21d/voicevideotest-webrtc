<template>
  <Page v-model:users="users" @update:users="update" />
</template>

<script>
import Page from '@/components/Page.vue'

export default {
  name: 'App',
  components: {
    Page
  },
  methods: {
    update() {
      // console.log(this.users)
    }
  },
  data() {
    return {
      users: [
        {
          name: 'Пётр Петрович',
          type: 'ученик',
        },
        {
          name: 'Николай Николаевич',
          type: 'ученик',
        },
        {
          name: 'Иван Иванович',
          type: 'ученик',
        },
        {
          name: 'Михаил Михайлович',
          type: 'учитель',
        },
      ],
    };
  }
}
</script>

<style>

</style>
