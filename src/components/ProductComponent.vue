<template>
  <div class="row q-pt-md">
    <div class="col-2 q-pr-md">
      <!-- filter by status here -->
      <q-select
        v-model="selectedStatus"
        :options="statusOptions"
        label="Select Status"
        standout="bg-primary text-white"
      />
    </div>
    <div class="col-2">
      <q-btn label="Confirm" color="primary" />
    </div>
  </div>
  <div align="right">
    <q-btn color="positive" label="Add" @click="addRecord()" />
  </div>

  <div v-if="dataReady" class="q-pt-md">
    <q-table
      style="height: 400px"
      flat
      bordered
      :rows="filteredRows"
      :columns="columns"
      row-key="id"
      virtual-scroll
      v-model:pagination="pagination"
      :rows-per-page-options="[0]"
      no-data-label="No data in product table"
      hide-bottom
    >
      <template #body="props">
        <q-tr :props="props">
          <q-td key="id" :props="props"> {{ props.row.id }}</q-td>
          <q-td key="pname" :props="props"> {{ props.row.pname }}</q-td>
          <q-td key="amount" :props="props"> {{ props.row.amount }}</q-td>
          <q-td key="descript" :props="props"> {{ props.row.descript }}</q-td>
          <q-td key="status" :props="props">
            <q-badge :color="getStatusColor(props.row.status)">
              {{ getStatusText(props.row.status) }}
            </q-badge>
          </q-td>
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
        <div class="text-h6">Add Product</div>
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
        <div class="text-subtitle1 text-weight-medium">Product:</div>
        <div>
          <q-input
            standout="bg-primary text-white"
            v-model="pname"
            type="text"
            label="Product name"
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
          />
        </div>

        <div class="text-subtitle1 text-weight-medium">Status:</div>
        <div>
          <q-select
            v-model="status"
            :options="statusOptions"
            label="Select Status"
            standout="bg-primary text-white"
            lazy-rules
            :rules="[requiredValidate]"
          />
        </div>
      </q-card-section>

      <q-card-actions align="right">
        <q-btn label="Add" color="positive" v-close-popup @click="onSubmit()" />
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
        <div class="text-h6">Edit Product Id: {{ input.id }}</div>
        <q-space />
        <q-btn color="negative" icon="close" flat round dense v-close-popup />
      </q-card-section>

      <q-card-section>
        <div class="text-subtitle1 text-weight-medium">Product:</div>
        <div>
          <q-input
            standout="bg-primary text-white"
            v-model="input.pname"
            type="text"
            label="Product name"
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
            label="Descript"
            @keyup.enter="onEdit()"
          />
        </div>

        <div class="text-subtitle1 text-weight-medium">Status:</div>
        <div>
          <q-select
            v-model="input.status"
            :options="statusOptions"
            label="Select Status"
            standout="bg-primary text-white"
            lazy-rules
            :rules="[requiredValidate]"
          />
        </div>
      </q-card-section>

      <q-card-actions align="right">
        <q-btn label="Save" color="positive" v-close-popup @click="onEdit()" />
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
        <span class="q-ml-sm">Delete product ID: {{ input.id }}</span>
      </q-card-section>

      <q-card-section>
        <span class="q-ml-sm">Product: {{ input.pname }}</span>
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

  <DialogComponent v-model="showDialog" :propDialog="dialog"></DialogComponent>
</template>

<script>
import { useLoginUserStore } from "../stores/loginUserStrore";
import { requiredValidate } from "../utils/validations";
import DialogComponent from "src/components/DialogComponent.vue";
import { Notify } from "quasar";

export default {
  name: "ProductComponent",
  data() {
    return {
      dataReady: false,
      form_add: false,
      form_edit: false,
      form_delete: false,
      admin_id: null,
      pname: null,
      amount: null,
      descript: null,
      status: null,
      statusOptions: [
        { label: "Pending", value: 1, color: "warning" },
        { label: "Finish", value: 2, color: "positive" },
        { label: "Not finish", value: 3, color: "negative" },
      ],
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
          name: "pname",
          label: "Product",
          field: "pname",
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
        {
          name: "status",
          label: "Status",
          field: "status",
          align: "left",
          sortable: true,
          format: (val) => this.getStatusText(val),
          style: (props) => ({
            color: props.row ? this.getStatusColor(props.row.status) : "black",
          }),
        },
        { name: "action", label: "Actions", field: "action", align: "center" },
      ],
      storeLogUser: useLoginUserStore(),
      input: [],
      showDialog: false,
      dialog: {
        icon: "",
        msg: "",
        btnType: "",
        iconColor: "primary",
      },
      selectedStatus: null,
      filterConfirmed: false,
    };
  },
  methods: {
    requiredValidate,
    addRecord() {
      this.form_add = true;
    },
    editRecord(record) {
      this.input = { ...record };
      this.input.status = this.statusOptions.find(
        (opt) => opt.value === record.status
      );
      this.form_edit = true;
    },
    deleteRecord(record) {
      this.input = record;
      this.form_delete = true;
    },
    getAllItems() {
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .get("/product/", { headers })
        .then((res) => {
          if (res.status == 200) {
            this.rows = res.data;
            this.confirmFilter();
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
    onSubmit() {
      const data = {
        admin_id: parseInt(this.storeLogUser.adminid),
        pname: this.pname,
        amount: this.amount,
        descript: this.descript,
        status: this.status.value,
      };
      console.log(JSON.stringify(data));
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .post("/product/create", data, { headers })
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
              "</div>Your product has been created successfully.";
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
        status: this.input.status.value,
      };

      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };

      this.$api
        .put(`/product/${this.input.id}`, data, { headers })
        .then((response) => {
          if (response.status === 200) {
            Notify.create({
              type: "positive",
              message: `Product ID ${response.data.id} updated successfully`,
            });
            this.form_edit = false;
            this.getAllItems();
          }
        })
        .catch((err) => {
          console.error(err);
          Notify.create({
            type: "negative",
            message: "Error updating product",
          });
        });
    },
    onDelete() {
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };

      this.$api
        .delete(`/product/${this.input.id}`, { headers })
        .then((response) => {
          if (response.status === 200) {
            Notify.create({
              type: "positive",
              message: `Product ID ${this.input.id} deleted successfully`,
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
      this.pname = null;
      this.amount = null;
      this.descript = null;
      this.status = null;
    },
    showErrDialog(err) {
      this.showDialog = true;
      this.dialog.icon = "error";
      this.dialog.iconColor = "negative";
      this.dialog.msg = err;
      this.dialog.btnType = "Error";
    },
    confirmFilter() {
      this.filterConfirmed = true;
      this.getAllItems();
    },
  },
  async mounted() {
    await this.getAllItems();
    this.dataReady = true;
  },
  computed: {
    getStatusText() {
      return (status) => {
        const selectedStatus = this.statusOptions.find(
          (opt) => opt.value === status
        );
        return selectedStatus ? selectedStatus.label : "Unknown";
      };
    },
    getStatusColor() {
      return (status) => {
        const selectedStatus = this.statusOptions.find(
          (opt) => opt.value === status
        );
        return selectedStatus ? selectedStatus.color : "black";
      };
    },
    filteredRows() {
      if (this.filterConfirmed && this.selectedStatus) {
        return this.rows.filter(
          (row) => row.status === this.selectedStatus.value
        );
      }
      return this.rows;
    },
  },
  components: { DialogComponent },
};
</script>
