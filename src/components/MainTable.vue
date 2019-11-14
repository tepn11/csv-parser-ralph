<template>
  <div class="q-pa-md fit">
    <div class="row">
      <div class="col"></div>
      <div class="col col-8">
        <div class="q-ma-lg">
          <div class="q-ma-md">
            <div>Load file</div>
            <vue-dropzone
              id="drop1"
              :options="dropOptions"
              @vdropzone-file-added="fileAdded"
            ></vue-dropzone>
          </div>
          <q-table
            class="q-ma-md"
            title="Trainings"
            :data="trainingData"
            :columns="trainingColumns"
            separator="cell"
            row-key="name"
          />
          <div class="q-ma-md">
            <q-btn
              :disable="data.length === 0"
              @click="showAddUser = true"
            >Add users</q-btn>
            <q-btn @click="showAddCourse = true">Add course</q-btn>
            <q-btn @click="showSettings = true">Settings</q-btn>
          </div>
          <q-dialog
            v-model="showAddCourse"
            persistent
          >
            <q-card style="min-width: 350px">
              <q-card-section>
                <div class="text-h6">lhq_Id list</div>
                <div class="text-caption">eg. '123,111,222'</div>
              </q-card-section>

              <q-card-section>
                <q-input
                  dense
                  v-model="lhqIds"
                  label="lhq_ids"
                  autofocus
                />
              </q-card-section>

              <q-card-actions
                align="right"
                class="text-primary"
              >
                <q-btn
                  flat
                  label="Cancel"
                  v-close-popup
                />
                <q-btn
                  flat
                  label="Add courses"
                  @click="addCourses()"
                  v-close-popup
                />
              </q-card-actions>
            </q-card>
          </q-dialog>
          <q-dialog
            v-model="showAddUser"
            persistent
          >
            <q-card style="min-width: 350px">
              <q-card-section>
                <div class="text-h6">Users</div>
                <div class="text-caption">eg. 'user1,user2,user3'</div>
              </q-card-section>

              <q-card-section>
                <q-input
                  dense
                  v-model="users"
                  label="trainee_login"
                  autofocus
                />
              </q-card-section>

              <q-card-actions
                align="right"
                class="text-primary"
              >
                <q-btn
                  flat
                  label="Cancel"
                  v-close-popup
                />
                <q-btn
                  flat
                  label="Add users"
                  @click="addUsers()"
                  v-close-popup
                />
              </q-card-actions>
            </q-card>
          </q-dialog>
          <q-dialog
            v-model="showSettings"
            persistent
          >
            <q-card style="min-width: 350px">
              <q-card-section>
                <div class="text-h6">Settings</div>
              </q-card-section>

              <q-card-section>
                <q-input
                  v-model="userIndex"
                  label="User column number"
                />
                <q-input
                  v-model="courseIndex"
                  label="Course column number"
                />
                <q-input
                  v-model="sheetName"
                  label="Sheet name"
                />
              </q-card-section>

              <q-card-actions
                align="right"
                class="text-primary"
              >
                <q-btn
                  flat
                  label="Cancel"
                  v-close-popup
                />
                <q-btn
                  flat
                  label="Save"
                  @click="saveSettings()"
                  v-close-popup
                />
              </q-card-actions>
            </q-card>
          </q-dialog>
          <q-dialog
            v-model="showError"
            persistent
          >
            <q-card style="min-width: 350px">
              <q-card-section>
                <div class="text-h6">Error</div>
              </q-card-section>

              <q-card-section>
                <div>{{ errorMsg }}</div>
              </q-card-section>

              <q-card-actions
                align="right"
                class="text-primary"
              >
                <q-btn
                  flat
                  label="Close"
                  v-close-popup
                />
              </q-card-actions>
            </q-card>
          </q-dialog>
        </div>
      </div>
      <div class="col"></div>
    </div>
  </div>
</template>
<script>
import vueDropzone from 'vue2-dropzone';
import 'vue2-dropzone/dist/vue2Dropzone.min.css';
import XLSX from 'xlsx';

export default {
  name: 'MainTable',
  components: {
    vueDropzone,
  },
  data() {
    return {
      showAddCourse: false,
      showAddUser: false,
      showSettings: false,
      showError: false,
      errorMsg: '',
      trainingColumns_dep: [
        {
          name: 'name',
          required: true,
          label: 'User',
          align: 'left',
          field: row => row.name,
          format: val => `${val}`,
          sortable: true,
        },
      ],
      trainingColumns: [
        {
          name: 'name',
          required: true,
          label: 'User',
          align: 'left',
          field: row => row[this.userIndex],
          format: val => `${val}`,
          sortable: true,
        },
      ],
      trainingData: [],
      users: '',
      lhqIds: '',
      data: [],
      dropOptions: {
        url: 'localhost',
        autoQueue: false,
        autoProcessQueue: false,
        addRemoveLinks: true,
      },
      courseIndex: 1,
      userIndex: 6,
      sheetName: 'Sheet1',
    };
  },
  methods: {
    addUsers() {
      const usersArray = this.users.split(',');
      usersArray.forEach((userId) => {
        this.addUser(userId.trim());
      });
      this.users = '';
    },
    addUser(userId) {
      const newUser = this.data[this.sheetName].filter(data => data[this.userIndex] === userId).reduce((acc, curr) => {
        if (!acc[this.userIndex]) {
          acc[this.userIndex] = curr[this.userIndex];
        }
        acc[curr[this.courseIndex]] = true;
        return acc;
      }, {});
      if (newUser[this.userIndex]) {
        this.trainingData.push(newUser);
      } else {
        this.showError = true;
        this.errorMsg = `Error: User ${userId} not found!`;
      }
    },
    addCourses() {
      const lhqIdsArray = this.lhqIds.split(',');
      lhqIdsArray.forEach((lhqId) => {
        this.addCourse(lhqId.trim());
      });
      this.lhqIds = '';
    },
    addCourse(lhqId) {
      const newCourse = {
        name: lhqId, label: lhqId, field: lhqId, format: val => `${val ? 'done' : ''}`,
      };
      this.trainingColumns.push(newCourse);
    },
    saveSettings() {
      return true;
    },
    fileAdded(file) {
      this.fileReader(file);
    },
    fileReader(file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        let data = e.target.result;
        data = new Uint8Array(data);
        const workbook = XLSX.read(data, { type: 'array' });
        const result = {};
        workbook.SheetNames.forEach((sheetName) => {
          const roa = XLSX.utils.sheet_to_json(workbook.Sheets[sheetName], { header: 1 });
          if (roa.length) result[sheetName] = roa;
        });
        // see the result, caution: it works after reader event is done.
        // console.log(result);
        this.data = result;
      };
      reader.readAsArrayBuffer(file);
    },
  },
};
</script>
