<template>
  <div>
    <div align="right">
      <q-btn color="positive" label="Add" @click="addRecord()" />
    </div>
    <div v-if="dataReady" class="q-pt-md">
      <q-table
        style="height: 400px"
        flat
        bordered
        :rows="rows"
        :columns="columns"
        row-key="id"
        virtual-scroll
        v-model:pagination="pagination"
        :rows-per-page-options="[0]"
        no-data-label="No data in user table"
        hide-bottom
      >
        <template #body="props">
          <q-tr :props="props">
            <q-td key="id" :props="props"> {{ props.row.id }}</q-td>
            <q-td key="fullname" :props="props"> {{ props.row.fullname }}</q-td>
            <q-td key="email" :props="props"> {{ props.row.email }}</q-td>
            <q-td key="password" :props="props"> {{ props.row.password }}</q-td>
            <q-td key="action" align="center">
              <q-btn
                color="positive"
                flat
                round
                icon="edit"
                @click="editRecord(props.row)"
              />
              <q-btn
                color="negative"
                flat
                round
                icon="delete"
                @click="deleteRecord(props.row)"
              />
            </q-td>
          </q-tr>
        </template>
      </q-table>
    </div>
    <div v-else class="q-pt-md" align="center">
      <q-circular-progress
        indeterminate
        rounded=""
        size="50px"
        color="blue-grey-4"
        class="q-ma-md"
      />
    </div>

    <!-- Add Modal Form -->
    <q-dialog v-model="form_add" persistent>
      <q-card>
        <q-card-section class="row items-center q-pb-none">
          <div class="text-h6">Add User</div>
          <q-space />
          <q-btn
            color="negative"
            icon="close"
            flat
            round
            dense
            v-close-popup
            @click="onReset()"
          />
        </q-card-section>

        <q-card-section>
          <div class="text-subtitle1 text-weight-medium">Fullname:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="fullname"
              type="text"
              label="Fullname"
              lazy-rules
              :rules="[requiredValidate]"
            />
          </div>

          <div class="text-subtitle1 text-weight-medium">Username:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="username"
              type="text"
              label="Username"
              lazy-rules
              :rules="[requiredValidate]"
            />
          </div>

          <div class="text-subtitle1 text-weight-medium">Email:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="email"
              type="text"
              label="Email"
              lazy-rules
              :rules="[emailValidate, requiredValidate]"
            />
          </div>

          <div class="text-subtitle1 text-weight-medium">Password:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="password"
              type="text"
              label="Password"
              lazy-rules
              :rules="[requiredValidate]"
            />
          </div>

          <div class="q-pt-md">
            <q-img
              v-if="isShowIcon"
              src="../../public/default-avatar.png"
              :ratio="4 / 3"
              spinner-color="primary"
              spinner-size="82px"
            />
            <q-img
              v-else
              :src="imageUrl"
              :ratio="4 / 3"
              spinner-color="primary"
              spinner-size="82px"
            />
            <q-file
              standout="bg-primary text-white"
              label="User avatar"
              v-model="upload_avatar"
              accept=".jpg, .jpeg, .png"
              max-file-size="1048576"
              @rejected="onRejected"
              @update:model-value="addFile()"
              class="q-pt-sm"
              lazy-rules
              :rules="[requiredValidate]"
            >
              <template v-slot:append>
                <q-icon name="attach_file" />
              </template>
            </q-file>
          </div>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn
            label="Add"
            color="positive"
            v-close-popup
            @click="onSubmit()"
          />
          <q-btn
            flat
            label="Cancel"
            color="negative"
            v-close-popup
            @click="onReset()"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Edit Modal Form -->
    <q-dialog v-model="form_edit" persistent>
      <q-card>
        <q-card-section class="row items-center q-pb-none">
          <div class="text-h6">Edit User Id: {{ input.id }}</div>
          <q-space />
          <q-btn color="negative" icon="close" flat round dense v-close-popup />
        </q-card-section>

        <q-card-section>
          <div class="text-subtitle1 text-weight-medium">Fullname:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="input.fullname"
              type="text"
              label="Fullname"
            />
          </div>

          <div class="text-subtitle1 text-weight-medium q-pt-md">Email:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="input.email"
              type="text"
              label="Email"
            />
          </div>

          <div class="text-subtitle1 text-weight-medium q-pt-md">Password:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="input.password"
              type="text"
              label="Password"
            />
          </div>

          <div class="q-pt-md">
            <q-img
              v-if="input.img"
              :src="input.img"
              :ratio="4 / 3"
              spinner-color="primary"
              spinner-size="82px"
            ></q-img>
            <!-- file size = 1MB -->
            <q-file
              standout="bg-primary text-white"
              label="User avatar"
              v-model="uploadFile"
              accept=".jpg, .jpeg, .png"
              max-file-size="1048576"
              @rejected="onRejected"
              @update:model-value="updateFile()"
              class="q-pt-sm"
            >
              <template v-slot:append>
                <q-icon name="attach_file" />
              </template>
            </q-file>
          </div>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn
            label="Save"
            color="positive"
            v-close-popup
            @click="onEdit()"
          />
          <q-btn
            label="Cancel"
            color="negative"
            v-close-popup
            @click="onCancelEdit()"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Delete Dialog -->
    <q-dialog v-model="form_delete" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="delete" color="primary" text-color="white" />
          <span class="q-ml-sm">Delete user ID: {{ input.id }}</span>
        </q-card-section>

        <q-card-section>
          <span class="q-ml-sm">Fullname: {{ input.fullname }}</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancel" color="positive" v-close-popup />
          <q-btn
            label="Delete"
            color="negative"
            v-close-popup
            @click="onDelete()"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <DialogComponent
      v-model="showDialog"
      :propDialog="dialog"
    ></DialogComponent>
  </div>
</template>

<script>
import { useLoginUserStore } from "../stores/loginUserStrore";
import { emailValidate, requiredValidate } from "../utils/validations";
import DialogComponent from "src/components/DialogComponent.vue";
import { Notify } from "quasar";
export default {
  name: "UserComponent",
  data() {
    return {
      dataReady: false,
      admin_id: null,
      username: null,
      fullname: null,
      email: null,
      password: null,
      upload_avatar: null,
      isShowIcon: true,
      imageUrl: "",
      rows: [],
      columns: [
        {
          name: "id",
          label: "ID",
          field: "id",
          align: "center",
          sortable: true,
        },
        {
          name: "fullname",
          label: "FullName",
          field: "fullname",
          align: "left",
          sortable: true,
        },
        {
          name: "email",
          label: "Email",
          field: "email",
          align: "left",
          sortable: true,
        },
        {
          name: "password",
          label: "Password",
          field: "password",
          align: "left",
          sortable: true,
        },
        { name: "action", label: "Actions", field: "action", align: "center" },
      ],
      storeLogUser: useLoginUserStore(),
      form_edit: false,
      form_add: false,
      form_delete: false,
      input: [],
      uploadFile: null,
      showDialog: false,
      dialog: {
        icon: "",
        msg: "",
        btnType: "",
        iconColor: "primary",
      },
    };
  },
  methods: {
    emailValidate,
    requiredValidate,
    getAllUsers() {
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .get("/user/", { headers })
        .then((res) => {
          if (res.status == 200) {
            res.data.forEach((item, key) => {
              if (item.img != null) {
                res.data[key].img =
                  this.$RESTAPI + "/file/" + res.data[key].img;
              }
            });
            this.rows = res.data;
          }
        })
        .catch((err) => {
          console.log(err);
          Notify.create({
            type: "negative",
            message: "Unauthorized",
          });
          this.storeLogUser.clearStorage();
          this.$router.push("/");
        });
    },
    addRecord() {
      this.form_add = true;
    },
    editRecord(record) {
      this.input = record;
      this.form_edit = true;
    },
    deleteRecord(record) {
      this.input = record;
      this.form_delete = true;
    },
    onRejected(rejectedEntries) {
      let msg;
      if (rejectedEntries[0].failedPropValidation == "accept")
        msg = "Only images (jpg, jpeg, png) are allowed";
      else if (rejectedEntries[0].failedPropValidation == "max-file-size")
        msg = "File size cannot be larger than 1MB";
      Notify.create({
        type: "negative",
        message: msg,
      });
    },
    addFile() {
      this.isShowIcon = false;
      this.imageUrl = URL.createObjectURL(this.upload_avatar);
    },
    updateFile() {
      this.input.img = URL.createObjectURL(this.uploadFile);
    },
    onSubmit() {
      // 1. upload file API
      // 2. user registration API
      if (this.upload_avatar == "") this.upload_avatar = null;
      if (this.upload_avatar) {
        //call upload file API
        const headers = {
          "Content-Type": "multipart/form-data",
        };
        const formData = new FormData();
        formData.append("singlefile", this.upload_avatar);
        this.$api
          .post("/file/upload", formData, { headers })
          .then((response) => {
            if (response.status == 200) {
              this.submitData(response.data.uploadFileName);
              this.getAllUsers();
              this.onReset();
            }
          })
          .catch((err) => {
            console.log(err);
            this.showErrorDialog(err);
          });
      } else {
        this.submitData(null);
        this.getAllUsers();
        this.onReset();
      }
    },
    submitData(filename) {
      const data = {
        admin_id: parseInt(this.storeLogUser.adminid),
        username: this.username,
        fullname: this.fullname,
        email: this.email,
        password: this.password,
        img: filename,
      };
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      console.log(JSON.stringify(data));
      this.$api
        .post("/user/create", data, { headers })
        .then((response) => {
          if (response.status == 200) {
            this.showDialog = true;
            this.dialog.icon = "task_alt";
            this.dialog.iconColor = "primary";
            this.dialog.msg =
              "<div class='text-h6'>Created, <br>" +
              response.data.fullname +
              "</div>Your user has been created successfully.";
            this.dialog.btnType = "Signup";
          }
          // this.$refs.registerForm.reset();
        })
        .catch((err) => {
          console.log(err);
          this.showErrorDialog(err);
        });
    },
    onCancelEdit() {
      this.getAllUsers();
    },
    onEdit() {
      // console.log("this is onEdit method");
      if (this.uploadFile == "") this.uploadFile = null;
      if (this.uploadFile) {
        // upload new image
        const headers = {
          "Content-Type": "multipart/form-data",
        };
        const formData = new FormData();
        formData.append("singlefile", this.uploadFile);
        this.$api
          .post("/file/upload", formData, { headers })
          .then((response) => {
            if (response.status == 200) {
              // call user edit API
              this.submitEditData(response.data.uploadFileName);
              this.updateFile = null;
            }
          })
          .catch((err) => {
            console.log(err);
            this.showErrDialog(err);
          });
      } else {
        // NOT upload any images
        this.submitEditData();
      }
      this.getAllUsers;
    },
    getFileName() {
      if (this.input.img) {
        return this.input.img.substr(this.input.img.lastIndexOf("/") + 1);
      }
      return null;
    },
    submitEditData(filename) {
      let img = "";
      if (filename == null) {
        if (this.input.img == null) img = null;
        else img = this.getFileName(this.input.img);
      } else img = filename;
      const data = {
        admin_id: this.storeLogUser.adminid,
        fullname: this.input.fullname,
        email: this.input.email,
        img: img,
      };
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .put("/user/" + this.input.id, data, { headers })
        .then((res) => {
          if (res.status == 200) {
            Notify.create({
              type: "positive",
              message: "Updated user ID: " + res.data.id,
            });
            if (this.storeLogUser.adminid == res.data.admin_id) {
              this.storeLogUser.fullname = res.data.fullname;
            }
          }
        })
        .catch((err) => {
          console.log(err);
          this.showErrDialog(err);
        });
    },
    onDelete() {
      // console.log("This is onDelete method");
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .delete("/user/" + this.input.id, { headers })
        .then((res) => {
          if (res.status == 200) {
            Notify.create({
              type: "positive",
              message: "Deleted user ID: " + res.data.id,
            });
            this.getAllUsers();
          }
        })
        .catch((err) => {
          console.log(err);
          this.showErrDialog(err);
        });
    },
    onReset() {
      this.username = null;
      this.fullname = null;
      this.email = null;
      this.password = null;
      this.upload_avatar = null;
      this.isShowIcon = true;
      this.imageUrl = "";
    },
    showErrDialog(err) {
      this.showDialog = true;
      this.dialog.icon = "error";
      this.dialog.iconColor = "negative";
      this.dialog.msg = err;
      this.dialog.btnType = "Error";
    },
  },
  async mounted() {
    await this.getAllUsers();
    this.dataReady = true;
  },
  components: { DialogComponent },
};
</script>
