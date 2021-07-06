<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-img
          :src="require('../assets/logo.svg')"
          class="my-3"
          contain
          height="200"
        />
      </v-col>

    </v-row>
   <input type="file" @change="onFileChange" />

  <!-- <v-textarea v-model="audits"></v-textarea> -->

  </v-container>
</template>

<script src="node_modules/pouchdb/dist/pouchdb.min.js"></script>
<script>
import readXlsxFile from 'read-excel-file'
import audits from '../data/audit-plan-import.txt'
import PouchDB from 'pouchdb'

// var db = new PouchDB('kittens')
var db = new PouchDB('http://admin:admin@localhost:5984/kittens');


export default {
    name: 'HelloWorld',

    data: () => ({
      audits: audits
    }),
    created() {
      // console.log('AUDITS: ', this.audits)
      db.info()
    },

    methods: {
      onFileChange(event) {
        let xlsxfile = event.target.files ? event.target.files[0] : null;
        var cnt = 0
        readXlsxFile(xlsxfile).then((rows) => {
          // console.log("rows:", rows)
          var records = []
          rows.forEach(element => {
            console.log('CNT: ', cnt)
            // console.log('DATA: ', element)
            var record = { Item: element[0],  AuditPerQtr: element[1], Month: element[2], Week: element[3], Date: element[4], Municipality: element[5], RegisteringAuthority: element[6], Days: element[7], AuditType: element[8], AuditOfficer: element[9]}
            // console.log(record)
            records.push(record)
            // element.forEach(item => {
            //   console.log(item)
            // })
            cnt++
          })
          console.log(records)
          db.bulkDocs(records)
          .then(response => {
            console.info(response)
          })
          .catch(err => {
            console.log(' ERROR: ', err)
          })
        })
      }
    }
  }
</script>
