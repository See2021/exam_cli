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
            <q-td key="title" :props="props"> {{ props.row.title }}</q-td>
            <q-td key="amount" :props="props"> {{ props.row.amount }}</q-td>
            <q-td key="descript" :props="props"> {{ props.row.descript }}</q-td>
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
          <div class="text-h6">Add Category</div>
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
          <div class="text-subtitle1 text-weight-medium">Category:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="title"
              type="text"
              label="Category name"
              lazy-rules
              :rules="[requiredValidate]"
            />
          </div>

          <div class="text-subtitle1 text-weight-medium">Amount:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="amount"
              type="text"
              label="Amount"
              lazy-rules
              :rules="[requiredValidate]"
            />
          </div>

          <div class="text-subtitle1 text-weight-medium">Descript:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="descript"
              type="text"
              label="descript"
              lazy-rules
              :rules="[requiredValidate]"
              @keyup.enter="onSubmit()"
            />
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
          <div class="text-h6">Edit Category Id: {{ input.id }}</div>
          <q-space />
          <q-btn color="negative" icon="close" flat round dense v-close-popup />
        </q-card-section>

        <q-card-section>
          <div class="text-subtitle1 text-weight-medium">Category:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="input.title"
              type="text"
              label="Category name"
              @keyup.enter="onEdit()"
            />
          </div>

          <div class="text-subtitle1 text-weight-medium">Amount:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="input.amount"
              type="text"
              label="Amount"
              @keyup.enter="onEdit()"
            />
          </div>

          <div class="text-subtitle1 text-weight-medium">Descript:</div>
          <div>
            <q-input
              standout="bg-primary text-white"
              v-model="input.descript"
              type="text"
              label="descript"
              @keyup.enter="onEdit()"
            />
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
            flat
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
          <span class="q-ml-sm">Delete category ID: {{ input.id }}</span>
        </q-card-section>

        <q-card-section>
          <span class="q-ml-sm">Title: {{ input.title }}</span>
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
  name: "ItemComponent",
  data() {
    return {
      dataReady: false,
      admin_id: null,
      title: null,
      amount: null,
      descript: null,
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
          name: "title",
          label: "Category",
          field: "title",
          align: "left",
          sortable: true,
        },
        {
          name: "amount",
          label: "Amount",
          field: "amount",
          align: "left",
          sortable: true,
        },
        {
          name: "descript",
          label: "Descript",
          field: "descript",
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
    getAllItems() {
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .get("/category/", { headers })
        .then((res) => {
          if (res.status == 200) {
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
    onSubmit() {
      const data = {
        admin_id: parseInt(this.storeLogUser.adminid),
        title: this.title,
        amount: this.amount,
        descript: this.descript,
      };
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .post("/category/create", data, { headers })
        .then((response) => {
          if (response.status === 200) {
            Notify.create({
              type: "positive",
              message: "Category added successfully",
            });
            this.showDialog = true;
            this.dialog.icon = "task_alt";
            this.dialog.iconColor = "primary";
            this.dialog.msg =
              "<div class='text-h6'>Created, <br>" +
              response.data.title +
              "</div>Your category has been created successfully.";
            this.dialog.btnType = "Signup";
            this.form_add = false;
            this.getAllItems();
            this.onReset();
          }
        })
        .catch((err) => {
          console.error(err);
          this.showErrorDialog(err);
          this.getAllItems();
          this.onReset();
        });
    },
    onCancelEdit() {
      this.getAllItems();
    },
    onEdit() {
      const data = {
        title: this.input.title,
        amount: this.input.amount,
        descript: this.input.descript,
      };

      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };

      this.$api
        .put(`/category/${this.input.id}`, data, { headers })
        .then((response) => {
          if (response.status === 200) {
            Notify.create({
              type: "positive",
              message: `Category ID ${response.data.id} updated successfully`,
            });
            this.form_edit = false;
            this.getAllItems();
          }
        })
        .catch((err) => {
          console.error(err);
          Notify.create({
            type: "negative",
            message: "Error updating category",
          });
        });
    },
    onDelete() {
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };

      this.$api
        .delete(`/category/${this.input.id}`, { headers })
        .then((response) => {
          if (response.status === 200) {
            Notify.create({
              type: "positive",
              message: `Category ID ${this.input.id} deleted successfully`,
            });
            this.form_delete = false;
            this.getAllItems();
          }
        })
        .catch((err) => {
          console.error(err);
          this.showErrDialog(err);
        });
    },
    onReset() {
      this.title = null;
      this.amount = null;
      this.descript = null;
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
    await this.getAllItems();
    this.dataReady = true;
  },
  components: { DialogComponent },
};
</script>
