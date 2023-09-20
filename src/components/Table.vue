<template>
<div>
  <v-data-table
    :headers="headers"
    :items="employeeItem"
    sort-by="calories"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-title>จัดการข้อมูล</v-toolbar-title>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-btn
              color="primary"
              dark
              class="mb-2"
              @click="openDialog('add',defaultItem)"
            >
              เพิ่มข้อมูล
            </v-btn>
       
      </v-toolbar>
    </template>

    <template v-slot:[`item.role`]="{ item }">
      {{ item.role.name}}
    </template>

    <template v-slot:[`item.actions`]="{ item }">
    <v-btn small outlined
        class="mr-2"
        @click="openDialog('edit',item)" color="green" >
      <v-icon>
        mdi-pencil
      </v-icon>
    </v-btn >
      <v-btn small outlined
        @click="deleteItem(item)" color="red" class="ml-2" >
      <v-icon >
        mdi-delete
      </v-icon>
      </v-btn>
    </template>
    <template v-slot:no-data>
      <v-btn
        color="primary"
        @click="initialize"
      >
        Reset
      </v-btn>
    </template>
  </v-data-table>
   <v-dialog v-model="dialogCreate"
          max-width="500px"
        >
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col
                    cols="12"
                    sm="6"
                    md="6"
                  >
                    <v-text-field
                      v-model="firstname"
                      label="ชื่อ"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="6"
                  >
                    <v-text-field
                      v-model="lastname"
                      label="นามสกุล"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="6"
                  >
                    <v-text-field
                      v-model="salary"
                      label="เงินเดือน"
                    ></v-text-field>
                  </v-col>
                  <v-col
                    cols="12"
                    sm="6"
                    md="6"
                  >
                    <v-text-field
                      v-model="roles"
                      label="ตำแหน่ง"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue darken-1"
                text
                @click="close()"
              >
                ยกเลิก
              </v-btn>
              <v-btn
                color="blue darken-1"
                text
                @click="save(formTitle)"
              >
                บันทึกข้อมูล
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5">คุณต้องการลบข้อมูลนี้ในตารางใช่ หรือ ไม่</v-card-title>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete()">ยกเลิก</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm()">ตกลง</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
</div>
</template>
<script>
  export default {
    data: () => ({
      firstname:'',
      lastname:'',
      salary:'',
      roles:'',
      dialogCreate: false,
      dialogDelete: false,
      headers: [
        {
          text: 'ไอดี',
          align: 'start',
          sortable: false,
          value: 'id',
        },
        { text: 'ชื่อ', value: 'firstName' },
        { text: 'นาสกุล', value: 'lastName' },
        { text: 'เงินเดือน', value: 'salary' },
        { text: 'ตำแหน่ง', value: 'role' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      employeeItem: [],
      editedIndex: -1,
      editedItem: {
        name: '',
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0,
      },
      defaultItem: {
        name: '',
        calories: 0,
        fat: 0,
        carbs: 0,
        protein: 0,
      },
      formTitle: '',
      idEmployee: '',
      idForDelete: ''

    }),

    watch: {
      dialog (val) {
        val || this.close()
      },
      dialogDelete (val) {
        val || this.closeDelete()
      },
    },

    created () {
      this.initialize()
    },

    methods: {
      async initialize () {
        this.employeeItem = [] 
        try{
          var data = await this.axios.get('http://localhost:9000/employee')
          console.log('data employee ====>',data)
          this.employeeItem = data.data
        }catch (console) {

        }
      },
      openDialog (Action,item) {
            console.log(Action,item)
            this.formTitle=''
            if(Action === 'add'){
                this.dialogCreate= true
                this.formTitle = 'เพิ่มข้อมูล'
                
            }else{
                this.dialogCreate= true
                this.formTitle = 'แก้ไขข้อมูล'
                // this.editedIndex = this.desserts.indexOf(item)
                // this.editedItem = item
                this.firstname = item.firstName
                this.lastname = item.lastName
                this.salary = item.salary
                this.roles = item.role.name
                this.idEmployee = item.id
            }
            
        },
 async save (action) {
        if (action === 'เพิ่มข้อมูล') {

          var data = {
            firstName: this.firstname,
            lastName: this.lastname,
            salary: this.salary,
            role: {
              name:this.roles
            },
            skills:[
              {skill:''}
            ],
          }
          console.log('data after send ====>',data)
          try{
            var dataResponse = await this.axios.post('http://localhost:9000/employee', data)
            // console.log('dataResponse ====>', dataResponse)
            this.close()
            this.initialize()
          }catch(error){
            console.log(error.massage);
          }
        }else{
          try{
            var dataResponseEdit = await this.axios.post('http://localhost:9000/employee/' + this.idEmployee, data)
            console.log('dataResponse ====>', dataResponseEdit)
            this.close()
            this.initialize()
          }catch(error){
            console.log(error.massage);
          }
            Object.assign(this.employeeItem[this.editedIndex], this.editedItem)
        }
        this.close()
      },
      editItem (item) {
        // console.log(`item select`,item);
        // console.log(`idex`);
        // this.editedIndex = this.desserts.indexOf(item)
        // this.editedItem = Object.assign({}, item)
        // this.dialog = true
        this.idForDelete = item.id
        this.dialogDelete = true
      },
      
      deleteItem (item) {
        this.idForDelete = item.id
        this.dialogDelete = true
      },

      async deleteItemConfirm () {
        try{
          var response = await this.axios.delete('http://localhost:9000/employee/' + this.idForDelete)
          this.initialize()
        }catch(error){
          console.log(error.massage);
        }
        this.closeDelete()
      },

      close () {
        this.dialogCreate = false
        this.firstname = '',
        this.lastname = '',
        this.salary ='',
        this.roles =''
        },
        closeDelete () {
            this.dialogDelete = false
            this.editedItem =[]
            this.editedIndex = -1
        }

      },
      }
</script>