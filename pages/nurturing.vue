<template>
  <v-row>
    <v-col class="text-center">
      <NurturingField :monsterID="monsterID" :love="love"/>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'Nurturing',
  data: function () {
    return {
      love: 0,
      monsterID: "020"
    }
  }
}
</script>
