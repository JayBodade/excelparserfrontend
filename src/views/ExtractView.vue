<template>
     <div class="home">
    <v-container>
      <v-row>
        <v-col cols="12">
          <v-select
            :items="templateData"
            v-model="selectedTemplate"
            item-value="_id"
            item-title="templateName"
            density="comfortable"
            label="Select Template"
            :error="!templateData.length"
            :error-messages="!templateData.length ? 'No templates available' : ''"
          ></v-select>
        </v-col>
      </v-row>

      <v-row v-if="selectedTemplate" justify="center">
        <v-col cols="12" md="8">
          <v-card
            height="130"
            class="d-flex align-center justify-center flex-column"
            :class="{ 'dragover': isDragOver }"
            @dragover.prevent="isDragOver = true"
            @dragleave.prevent="isDragOver = false"
            @drop.prevent="handleDrop"
          >
            <v-icon size="55">mdi-upload</v-icon>
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

      <v-row v-if="Object.keys(extractedInfo.fieldData).length" class="mt-5">
        <v-col>
          <v-list>
            <v-list-item-group>
              <v-list-item
                v-for="(item, index) in Object.keys(extractedInfo.fieldData)"
                :key="index"
              >
                <v-list-item-content>
                  <v-list-item-title>{{ item }}</v-list-item-title>
                  <v-list-item-subtitle class="text-body-1">
                    {{ extractedInfo.fieldData[item] }}
                  </v-list-item-subtitle>
                </v-list-item-content>
              </v-list-item>
            </v-list-item-group>
          </v-list>
        </v-col>
      </v-row>

      <v-row v-if="extractedInfo.tableData?.tableHeader" justify="center" class="mt-5">
        <v-col cols="12">
          <h6>Table Data</h6>
          <v-simple-table class="data-table">
            <thead>
              <tr>
                <th v-for="(header, index) in extractedInfo.tableData.tableHeader" :key="index">
                  {{ header }}
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, rowIndex) in extractedInfo.tableData.data" :key="rowIndex">
                <td v-for="(cell, cellIndex) in row" :key="cellIndex">
                  {{ cell || '-' }}
                </td>
              </tr>
            </tbody>
          </v-simple-table>
        </v-col>
      </v-row>

      <v-row v-if="extractedInfo.summaryData?.name" class="mt-5">
        <v-col cols="12">
          <h6 class="mb-3">Summary Info</h6>
          <v-row>
            <v-col>
              <strong>Name:</strong> {{ extractedInfo.summaryData.name }}
            </v-col>
          </v-row>
          <v-divider class="my-3"></v-divider>
          <v-row v-if="Object.keys(extractedInfo.summaryData.data).length">
            <v-col>
              <v-row
                v-for="(value, key, index) in extractedInfo.summaryData.data"
                :key="index"
                class="py-1"
              >
                <v-col cols="6" class="font-weight-bold">
                  {{ key }}:
                </v-col>
                <v-col cols="6">
                  {{ value }}
                </v-col>
              </v-row>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>
<script>
import readXlsxFile from 'read-excel-file';
export default {
    data() {

        return {
            loading: false,
            templateData: [],
            selectedTemplate: null,
            file: null,
            extractedInfo: {
                fieldData: {},
                tableData: {},
                summaryData: {}
            },
        }

    },
    methods: {
        async getAllTemplates() {
            try {
                this.loading = true;
                let url = `${process.env.BACKEND_URL}/alltemplates`
                let response = await fetch(url, {
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
                this.loading = false;
            }
        },
        handleDrop(event) {
            this.file = event.dataTransfer.files[0];
            this.isDragOver = false;
            this.extractDataFromFile();


        },
        handleFileChange(event) {
            this.file = event.target.files[0];
            this.extractDataFromFile();
        },
        extractDataFromFile() {
            let template = (this.templateData || []).find((item) => item._id == this.selectedTemplate);
            console.log(template);
            readXlsxFile(this.file).then((rows) => {
                template.fieldData.forEach((ele) => {
                    this.extractedInfo.fieldData[ele.name] = rows[parseInt(ele.row) - 1][parseInt(ele.col) - 1];
                })

                this.extractedInfo.tableData.tableName = template.tableData.name;

                let tableRow = parseInt(template.tableData.row) - 1;
                console.log("this is row", tableRow);
                this.extractedInfo.tableData.tableHeader = rows[tableRow];
                let tableCol = parseInt(template.tableData.col) - 1;
                this.extractedInfo.tableData.data = this.getTableData(rows, tableRow, tableCol);
                console.log(this.extractedInfo);
                let summaryCol = parseInt(template.summary.colnumber) - 1;
                this.extractedInfo.summaryData.name = template.summary.name;
                this.extractedInfo.summaryData.data = this.getSummaryData(rows, this.extractedInfo.tableData.tableHeader, template.summary.text, summaryCol);

                console.log(this.extractedInfo);
            }).catch((e) => {
                console.log("this is error", e);
            })

        },
        checkForAllNull(arr) {
            let allIsNull = true;
            arr.forEach(element => {
                if (element != null) {
                    allIsNull = false
                }
            });

            return allIsNull;
        },
        getTableData(extractedRows, row, col) {
            console.log("running get table data fucntions", row, col);
            let resultarray = [];
            for (let i = row; ; i++) {
                if (i == extractedRows.length) break;
                if (this.checkForAllNull(extractedRows[i])) break;
                else {
                    if (extractedRows[i].length > col + 1) {
                        let len = extractedRows[i].length;
                        let elemsToDelete = len - col + 1;
                        let arr = extractedRows[i].splice(extractedRows[i].length - elemsToDelete,
                            elemsToDelete);
                        resultarray.push(arr);
                    } else {
                        resultarray.push(extractedRows[i]);
                    }
                }
            }


            return resultarray;
        },
        getSummaryData(extractedRows, header, text, col) {
            let rowIndex = this.getRowIndex(extractedRows, text, col);
            let resummary = {};
            if (rowIndex >= 0) {
                let totalRow = extractedRows[rowIndex];
                header.forEach((element, index) => {
                    console.log(element);
                    if (totalRow[index] == text) {
                        resummary[element] = '-';
                    } else {
                        resummary[element] = totalRow[index];
                    }
                });

            }

            return resummary;

        },
        getRowIndex(extractedRows, text, col) {

            for (let i = 0; i < extractedRows.length; i++) {
                if (extractedRows[i][col] == text) return i;
            }
            return -1;

        },
    },

    watch:{
        selectedTemplate(newVal,oldVal){
            if(newVal != oldVal){
                this.file = null;
                this.extractedInfo={
                fieldData: {},
                tableData: {},
                summaryData: {}
            }
            }
        }
    },
    created() {
        this.getAllTemplates();
    },
    computed: {
        fileSize() {
            if (!this.file) return '';
            const size = this.file.size / 1024 / 1024;
            return `${size.toFixed(2)} MB`;
        },
    },
}
</script>
<style scoped>
.home {
    margin-top: 4%;
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
    /* background-color: #4CAF50; */
    color: white;
  }
</style>