<template>
  <div class="home">
    <div v-if="templateLoading"   style="height:70vh;display: flex; justify-content: center; align-items: center;">
    <v-progress-circular
      color="primary"
      indeterminate
      size="70"
    ></v-progress-circular>
  </div>
    <v-data-table v-else  :headers="headers" :items="templateData || []" items-per-page="10" :search="search" item-value="_id" class="elevation-1 "
      density="compact" :loading="loading">
      <template v-slot:top>
        <v-row class="px-4">
          <v-col cols="10">
            <v-text-field v-model="search" class="pa-2" label="Search Template" outlined dense></v-text-field>
          </v-col>
          <v-col cols="2" class="d-flex justify-end align-center">
            <v-btn color="indigo darken-2" @click="$router.push('/addtemplate')">
              Add Template
            </v-btn>
          </v-col>
        </v-row>
      </template>

      <template v-slot:[`item.actions`]="{ item }">
        <v-btn @click="editItem(item)" icon flat> <v-icon color="indigo darken-2">mdi-file-edit-outline</v-icon></v-btn>
        <v-btn @click="deleteItem(item)" icon flat> <v-icon color="red">mdi-trash-can-outline</v-icon> </v-btn>
      </template>
    </v-data-table>

    <v-dialog v-model="dialog">
    <v-card class="mx-auto" width="600" style="padding: 0px 10px !important;">
      <v-form ref="updateFormRef">
      <div>
        <v-row justify="center py-2">
          <v-col cols="11">
            <h5>Template Name</h5>
            <v-row class="mb-4">
              <v-text-field  :rules="[v => !!v || 'Item is required']" color="primary" label="Template Name" v-model="popupData.templateName" variant="underlined" density="compact" class="mx-4"></v-text-field>
            </v-row>
            <h5>Field Data</h5>
            <div ref="scrollContainer" style="max-height: 100px; overflow-y: auto; overflow-x: hidden; background-color: #f6f5f5; border-radius: 10px; border: 1px solid #ddd;">
              <v-row v-for="(item, index) in popupData.fieldData" :key="index" class="mb-1 align-center" no-gutters>
                <v-col cols="3.6" class="px-2">
                  <v-text-field  :rules="[v => !!v || 'Item is required']" color="primary"  v-model="item.name" label="Field Name" variant="underlined" density="compact"></v-text-field>
                </v-col>
                <v-col cols="3.6" class="px-2">
                  <v-text-field  :rules="[v => !!v || 'Item is required']" color="primary" type="number" v-model="item.row" label="Row Number" variant="underlined" density="compact"></v-text-field>
                </v-col>
                <v-col cols="3.6" class="px-2">
                  <v-text-field  :rules="[v => !!v || 'Item is required']"  color="primary" type="number" v-model="item.col" label="Column Number" variant="underlined" density="compact"></v-text-field>
                </v-col>
                <v-col cols="1" class="px-2">
                 <v-btn icon flat @click="removeFieldItem(index)" > <v-icon color="red">mdi-close</v-icon> </v-btn>
                </v-col>
              </v-row>
            </div>
            <v-row justify="end" class="mt-3">
              <v-btn size="small" color="primary" @click="addRow()" class="mx-2">
                Add
              </v-btn>
            </v-row>

            <h5>Table Data</h5>
            <v-row class="mb-2">
              <v-col cols="4">
                <v-text-field  :rules="[v => !!v || 'Item is required']" v-model="popupData.tableData.name" color="primary" label="Table Name" variant="underlined" density="compact"></v-text-field>
              </v-col>
              <v-col cols="4">
                <v-text-field  :rules="[v => !!v || 'Item is required']" color="primary" type="number" v-model="popupData.tableData.row"  label="Row Number" variant="underlined" density="compact"></v-text-field>
              </v-col>
              <v-col cols="4">
                <v-text-field :rules="[v => !!v || 'Item is required']" color="primary" type="number" v-model="popupData.tableData.col"  label="Enter Number of Columns" variant="underlined" density="compact"></v-text-field>
              </v-col>
              
            </v-row>

            <h5>Summary Data</h5>
            <v-row>
              <v-col cols="4">
                <v-text-field  :rules="[v => !!v || 'Item is required']" color="primary" v-model="popupData.summary.name"  label="Summary Name" variant="underlined" density="compact"></v-text-field>
              </v-col>
              <v-col cols="4">
                <v-text-field  :rules="[v => !!v || 'Item is required']" color="primary" v-model="popupData.summary.text"  label="Summary Tezt" variant="underlined" density="compact"></v-text-field>
              </v-col>
              <v-col cols="4">
                <v-text-field  :rules="[v => !!v || 'Item is required']" color="primary" type="number" v-model="popupData.summary.colnumber" label="Column Number" variant="underlined" density="compact"></v-text-field>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </div>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn text="Save" color="indigo darken-2" @click="updateTemplate()"></v-btn>
        <v-btn text="Close" color="red" @click="dialog = false"></v-btn>
      </v-card-actions>
    </v-form>
    </v-card>
  </v-dialog>
  </div>
</template>

<script>
export default {
  name: 'HomeView',
  data() {
    return {
      templateLoading:false,
      loading: false,
      templateData: [],
      dialog: false,
      search: '',
      popupData: {},
      headers: [
        { title: 'Template Id', align: 'center', key: 'templateId' },
        { title: 'Template Name', align: 'center', key: 'templateName' },
        { title: 'Table Name', align: 'center', key: 'tableData.name' },
        { title: 'Summary', align: 'center', key: 'summary.name' },
        { title: 'Actions', align: 'center', key: 'actions', sortable: false },

      ]
    };
  },
  methods: {
    async getAllTemplates() {
      try {
        this.templateLoading = true;
        let url = process.env.VUE_APP_BACKEND_URL;
        let response = await fetch(`${url}/alltemplates`, {
          method: 'GET',
          headers: { 'content-type': 'application/json' }
        });

        let result = await response.json();
        console.log('this is result', result);
        if (result.success) {
          this.templateData = result.all_templates;
        }
      } catch (e) {
        console.log('something went wrong', e.message);
      } finally {
        this.templateLoading = false;
      }
    },
    editItem(item) {
      this.dialog = true;
      console.log("this is the item",item);
      this.popupData = JSON.parse(JSON.stringify(item)) 
    },
    async deleteItem(item) {

      try {
        this.loading = true;
        let template_Id = item._id;
        let url = `${process.env.VUE_APP_BACKEND_URL}/deletetemplate?template_Id=${template_Id}`;
        const response = await fetch(url, {
          method: 'DELETE',
          headers: {
            'content-type': 'application/json',
          }
        })
        console.log("this is template id", template_Id);

        let result = await response.json();
        if (result.success) {
          this.templateData = (this.templateData || []).filter((item) => item._id != template_Id);
        }

      } catch (e) {
        console.log("something went wrong while deleting template");
      } finally {
        this.loading = false;
      }

    },
    addRow(){
      (this.popupData.fieldData || []).push({
        name:'',
        row:'',
        col:'',
      })
    },
    removeFieldItem(i){
    this.popupData.fieldData = (this.popupData.fieldData || []).filter((item,index)=>index!=i);
  },
  async updateTemplate(){
  
    try{
      //updateFormRef
     
    if(this.popupData){
      const { valid } = await this.$refs.updateFormRef.validate()
      console.log("this is valid",valid);
        if(!valid){
          return;
        }else{
     
      const url = `${process.env.VUE_APP_BACKEND_URL}/updatetemplate?template_Id=${this.popupData._id}`;
      let data = {
        templateName:this.popupData.templateName,
        fieldData:this.popupData.fieldData,
        summary:this.popupData.summary,
        tableData:this.popupData.tableData,
      }

      console.log(data);
      const response = await fetch(url,{
        method:'PUT',
        headers:{
          'content-type':'application/json',
        },
        body:JSON.stringify({data:data})
      });

      const result = await response.json();
      console.log(result);
      if(result.success){
        let ele = this.templateData.find((item)=>item._id == this.popupData._id);
        ele.templateName = this.popupData.templateName;
        ele.fieldData = this.popupData.fieldData;
        ele.summary = this.popupData.summary;
        ele.tableData = this.popupData.tableData;
        this.dialog = false;
      }
    }
    }
  }catch(e){
    console.log("something went wrong",e);
  }

  },
  },
  
  created() {
    this.getAllTemplates();
  }
};
</script>

<style>
.home {

  width: 90%;
  margin: auto;
  margin-top: 6%;
}

.v-data-table {
  width: 100%;
  /* Ensure table takes full width */
}
</style>
