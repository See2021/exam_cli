<template>
  <q-page padding>
    <q-btn-dropdown
      color="grey-4"
      :label="selectedComponentLabel"
      class="q-mr-md text-capitalize text-black text-weight-regular"
    >
      <q-list>
        <q-item clickable v-close-popup @click="onItemClick(null)">
          <q-item-section>
            <q-item-label>Select Component</q-item-label>
          </q-item-section>
        </q-item>

        <q-item clickable v-close-popup @click="onItemClick('UserComponent')">
          <q-item-section>
            <q-item-label>User</q-item-label>
          </q-item-section>
        </q-item>

        <q-item clickable v-close-popup @click="onItemClick('ItemComponent')">
          <q-item-section>
            <q-item-label>Category</q-item-label>
          </q-item-section>
        </q-item>

        <q-item
          clickable
          v-close-popup
          @click="onItemClick('ProductComponent')"
        >
          <q-item-section>
            <q-item-label>Product</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-btn-dropdown>

    <q-btn
      color="primary"
      label="Search"
      @click="onSearchClick"
      class="text-capitalize"
    />

    <!-- Display the selected component only when the search button is clicked -->
    <component :is="selectedComponent" v-if="showComponent" class="q-pt-md" />
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import UserComponent from "src/components/UserComponent.vue";
import ItemComponent from "src/components/ItemComponent.vue";
import ProductComponent from "src/components/ProductComponent.vue";

export default defineComponent({
  name: "TestPage",
  components: { UserComponent, ItemComponent, ProductComponent },
  data() {
    return {
      selectedComponent: null,
      showComponent: false,
      componentLabels: {
        UserComponent: "User",
        ItemComponent: "Category",
        ProductComponent: "Product",
      },
    };
  },
  computed: {
    selectedComponentLabel() {
      return this.selectedComponent
        ? this.componentLabels[this.selectedComponent]
        : "Select Component";
    },
  },
  methods: {
    onItemClick(componentName) {
      this.selectedComponent = componentName;
      this.showComponent = false;
    },
    onSearchClick() {
      this.showComponent = true;
    },
  },
});
</script>
