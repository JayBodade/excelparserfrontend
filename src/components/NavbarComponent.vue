<template>
<v-card class="mx-auto" color="indigo darken-2" max-width="448" style="z-index: 5;">
    <v-layout>
    <v-app-bar color="indigo darken-2">
      <v-row align="center" no-gutters>
        <v-col class="d-flex justify-center" style="height: fit-content;">
          <v-img alt="Vuetify Name" contain width="100" src="https://res.cloudinary.com/incquet-solution/image/upload/v1580409178/Incquet%20website/zoho_logo_white_kinwcg.png" />
        </v-col>
        
        <v-spacer></v-spacer>
        
        <v-col class="d-flex justify-center">
          <v-tabs v-model="tab" align-tabs="center">
            <v-tab @click="$router.push('/')" value='/'>All Templates</v-tab>
            <v-tab @click="$router.push('/extractdata')" value='/extractdata'>Extract Data</v-tab>
          </v-tabs>
        </v-col>
        
        <v-spacer></v-spacer>
        
        <v-col class="d-flex justify-center">
          <v-img src="../assets/logo1.png" max-width="60" class="ml-2"></v-img>
        </v-col>
      </v-row>
    </v-app-bar>
</v-layout>
  </v-card>
</template>
<script>
export default{
    data(){
        return {
            tab:'/',
        }
    },
    created(){
        this.tab=window.location.pathname;
    }

}
</script>
<style >
.v-btn {
  font-size: 14px;
}
.v-app-bar {
  display: flex;
  align-items: center;
}


</style>
