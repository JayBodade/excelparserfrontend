<template>
    <v-container class="file-input-container" style="margin-top: 4%;">
      <v-row justify="center">
        <v-col cols="12" md="8">
          <v-card height="130" class="d-flex align-center justify-center flex-column  "
            :class="{ 'dragover': isDragOver }" @dragover.prevent="isDragOver = true"
            @dragleave.prevent="isDragOver = false" @drop.prevent="handleDrop">
            <v-icon size="55" color="indigo darken-2">mdi-upload</v-icon>
            <v-btn v-if="!file" flat @click="$refs.fileInput.click()" class="mt-4">
              Select File
            </v-btn>
            <input ref="fileInput" type="file" class="d-none" @change="handleFileChange" />
            <v-chip v-if="file" class="ma-2" close @click:close="file = null">
              {{ file.name }} - {{ fileSize }}
            </v-chip>
  
          </v-card>
        </v-col>
      </v-row>

      <v-form v-if="parsedData" ref="form">
  
      <v-row justify="center">
        <v-col cols="7">
          <h5>Template Name</h5>
          <v-row class="mb-4">
            <v-text-field color="primary" :rules="[v => !!v || 'Item is required']"  v-model="templateData.templateName" label="Template Name" variant="underlined"
              density="compact" class="mx-4"></v-text-field>
          </v-row>
          <h5>Field Data</h5>
          <div ref="scrollContainer"
            style="max-height: 100px; overflow-y: auto; overflow-x: hidden; background-color: #f6f5f5; border-radius: 10px; border: 1px solid #ddd;">
            <v-row v-for="(item, index) in templateData.fieldData" :key="index" class="mb-1 align-center" no-gutters>
              <v-col cols="3.6" class="px-2">
                <v-text-field color="primary" :rules="[v => !!v || 'Item is required']" v-model="item.name" label="Field Name" variant="underlined"
                  density="compact"></v-text-field>
              </v-col>
              <v-col cols="3.6" class="px-2">
                <v-text-field color="primary"  :rules="[v => !!v || 'Item is required']" type="number" v-model="item.row" label="Row Number" variant="underlined"
                  density="compact"></v-text-field>
              </v-col>
              <v-col cols="3.6" class="px-2">
                <v-text-field color="primary"  :rules="[v => !!v || 'Item is required']" type="number" v-model="item.col" label="Column Number" variant="underlined"
                  density="compact"></v-text-field>
              </v-col>
              <v-col cols="1" class="px-2">
                 <v-btn v-if="templateData.fieldData.length > 1" icon flat @click="removeFieldItem(index)" > <v-icon color="red">mdi-close</v-icon> </v-btn>
                </v-col>

            </v-row>
          </div>
          <v-row justify="end" class="mt-3">
            <v-btn size="small" color="indigo darken-2" @click="addRow()" class="mx-2">
              Add
            </v-btn>
          </v-row>
  
          <h5>Table Data</h5>
          <v-row class="mb-2">
            <v-col cols="4">
              <v-text-field color="primary" :rules="[v => !!v || 'Item is required']" v-model="templateData.tableData.name" label="Table Name" variant="underlined"
                density="compact"></v-text-field>
            </v-col>
            <v-col cols="4">
              <v-text-field color="primary" :rules="[v => !!v || 'Item is required']" type="number" v-model="templateData.tableData.row" label="Row Number" variant="underlined"
                density="compact"></v-text-field>
            </v-col>
            <v-col cols="4">
              <v-text-field color="primary" :rules="[v => !!v || 'Item is required']" type="number" v-model="templateData.tableData.col" label="Number of Columns"
                variant="underlined" density="compact"></v-text-field>
            </v-col>

          </v-row>
  
          <h5>Summary Data</h5>
          <v-row>
            <v-col cols="4">
              <v-text-field color="primary" :rules="[v => !!v || 'Item is required']" v-model="templateData.summary.name" label="Summary Name" variant="underlined"
                density="compact"></v-text-field>
            </v-col>
            <v-col cols="4">
              <v-text-field color="primary" :rules="[v => !!v || 'Item is required']" v-model="templateData.summary.text" label="Summary Text" variant="underlined"
                density="compact"></v-text-field>
            </v-col>
            <v-col cols="4">
              <v-text-field color="primary" :rules="[v => !!v || 'Item is required']" type="number" v-model="templateData.summary.colnumber" label="Column Number"
                variant="underlined" density="compact"></v-text-field>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
  
      <v-row justify="center">
        <v-col cols="8" class="text-center">
          <v-btn size="small" color="indigo darken-2" :loading="loading" class="mx-2" @click="addTemplateToDB">
            Save
          </v-btn>
        </v-col>
      </v-row>

    </v-form>
  
      <v-row v-if="parsedData" justify="center" class="mt-5">
        <v-col cols="12">
  
          <table class="data-table">
            <tr v-for="(row, rowIndex) in parsedData" :key="rowIndex">
              <td v-for="(cell, cellIndex) in row" :key="cellIndex">
                {{ cell || '-' }}
                <br>
                <span style="color: #666769;opacity: 0.8;font-size: 13px;">({{ rowIndex + 1 }},{{ cellIndex + 1 }})</span>
              </td>
            </tr>
          </table>
        </v-col>
      </v-row>
    </v-container>
  </template>
  
  <script>
  import readXlsxFile from 'read-excel-file';
  export default {
    data() {
      return {
        file: null,
        loading:false,
        isDragOver: false,
        parsedData: null,
        templateData: {
          templateName: '',
          fieldData: [
            {
              name: '',
              row: '',
              col: '',
            }
          ],
          tableData: {
            name: '',
            row: '',
            col: '',
          },
          summary: {
            name: '',
            text: '',
            colnumber: '',
  
          }
  
        }
      };
    },
    computed: {
      fileSize() {
        if (!this.file) return '';
        const size = this.file.size / 1024 / 1024;
        return `${size.toFixed(2)} MB`;
      },
    },
    methods: {
      handleFileChange(event) {
        this.file = event.target.files[0];
        readXlsxFile(this.file).then((rows) => {
          console.log(rows);
          this.parsedData = rows;
        }).catch((e) => {
          console.log(e);
        })
      },
      handleDrop(event) {
        this.file = event.dataTransfer.files[0];
        this.isDragOver = false;
        readXlsxFile(this.file).then((rows) => {
          console.log(rows);
          this.parsedData = rows;
        }).catch((e) => {
          console.log(e);
        })
  
      },
      removeFieldItem(i){
    this.templateData.fieldData = (this.templateData.fieldData || []).filter((item,index)=>index!=i);
  },  
      addRow() {
        this.templateData.fieldData.push({ name: '', row: '', col: '' })
      },
      async addTemplateToDB() {
        try {
           const { valid } = await this.$refs.form.validate()

        if (!valid){ 
          
        return;}
          console.log({data:this.templateData});
          this.loading = true;
          let url = `${process.env.VUE_APP_BACKEND_URL}/addtemplate`;
          const resp = await fetch(url, {
            method: 'POST',
            headers: {
              'content-type': 'application/json',
            },
            body: JSON.stringify({data:this.templateData})
          })
  
          const result = await resp.json();
          console.log(result);
          this.$router.push('/');
        } catch (e) {
          console.log("something went wrong", e);
        }finally{
          this.loading = false;
        }
  
  
      },
    }
  }
  </script>
  
  <style scoped>
  .file-input-container {
    margin-top: 0px;
  }
  
  .v-card {
    height: 200px;
    border: 2px dashed #ccc;
    transition: border-color 0.3s;
  }
  
  .v-card.dragover {
    border-color: #1976d2;
  }
  
  .v-card .v-icon {
    color: #1976d2;
  }
  
  .data-table {
    width: 100%;
    border-collapse: collapse;
    margin: 20px 0;
  }
  
  .data-table td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: center;
  }
  
  .data-table tr:nth-child(even) {
    background-color: #f2f2f2;
  }
  
  .data-table tr:hover {
    background-color: #ddd;
  }
  
  .data-table th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: center;
    background-color: #4CAF50;
    color: white;
  }
  </style>
  