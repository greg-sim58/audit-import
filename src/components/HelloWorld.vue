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

const dbName = 'auditplantemplate'

// var db = new PouchDB('kittens')
var db = new PouchDB(`http://admin:admin@localhost:5984/${dbName}`);


export default {
    name: 'HelloWorld',

    data: () => ({
      audits: audits
    }),
    created() {
      db.info()
    },

    methods: {
      onFileChange(event) {
        let xlsxfile = event.target.files ? event.target.files[0] : null;
        var cnt = 0
        readXlsxFile(xlsxfile).then((rows) => {
          const year = this.startYear(rows[0])
          var records = []
          rows.forEach(element => {
            var record = { Item: element[0],  AuditPerQtr: element[1], Year: year, Month: this.getMonth(element), Week: element[3], Date: element[4], StartDate: this.startDate(element), EndDate: this.endDate(element), Municipality: element[5], RegisteringAuthority: element[6], Days: element[7], AuditType: element[8], AuditOfficer: element[9]}
            console.info('RECORD: ', record)
            
            records.push(record)
            cnt++
          })
          console.log(records[0])
          db.bulkDocs(records)
          .then(response => {
            db.info()
          })
          .catch(err => {
            console.log(' ERROR: ', err)
          })
        })
      },
      startDate(auditItem) {
          var year = auditItem[2].split(/(\s+)/).filter( function(e) { return e.trim().length > 0; } )[1]
          // console.info('year: ', year)
          
        if (auditItem[4] !== null) {
          // split by spacer
          var allspace = auditItem[4].replace(/\-+/g, ' ')
          var splitspace = allspace.split(/(\s+)/).filter( function(e) { return e.trim().length > 0; } )

          if (splitspace.length == 3) {
            console.info('START DATE: ', splitspace[0] + ' ' + splitspace[2] + ' ' + year)
            return splitspace[0] + ' ' + splitspace[2] + ' ' + year
          } else {
            console.info('START DATE: ', splitspace[0] + ' ' + splitspace[1] + ' ' + year)
            return splitspace[0] + ' ' + splitspace[1] + ' ' + year
          }
        } else {
          return ''
        }
        
      },
      endDate(auditItem) {
                  var year = auditItem[2].split(/(\s+)/).filter( function(e) { return e.trim().length > 0; } )[1]
        if (auditItem[4] !== null) {
          var allspace = auditItem[4].replace(/\-+/g, ' ')
          var splitspace = allspace.split(/(\s+)/).filter( function(e) { return e.trim().length > 0; } )

          if (splitspace.length == 3) {
            var enddate =  splitspace[1] + ' ' + splitspace[2] + ' ' + year
            console.info('END DATE: ',enddate)
            return enddate
          } else {
            enddate = splitspace[2] + ' ' + splitspace[3] + ' ' + year
            console.info('END DATE: ', enddate)
            return enddate
          }
      }
 },
      getMonth(auditItem) {
        if (auditItem !== null) {
          return auditItem[2].split(/(\s+)/).filter( function(e) { return e.trim().length > 0; } )[0]
        } else {
          return ''
        }
      },
      startYear(auditItem) {
        if (auditItem[2] !== null) {
          return auditItem[2].split(/(\s+)/).filter( function(e) { return e.trim().length > 0; } )[1];          
        } else {
          return ''
        }
      }
    }
  }
</script>
