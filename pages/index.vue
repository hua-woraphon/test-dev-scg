<template>
  <UContainer>
    <UCard class="mt-10">
      <div class="flex justify-between px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
        <UInput size="lg" icon="i-heroicons-magnifying-glass-20-solid" autocomplete="off"
          :ui="{ icon: { trailing: { pointer: '' } } }" v-model="searchInput" :onchange="search()"
          placeholder="Filter Material..." />
        <UButton @click="isOpenModal = true" icon="i-heroicons-pencil-square" size="lg" color="primary" variant="solid"
          label="Add" :trailing="true" class="px-6" />
      </div>
      <UTable :style="'max-height: 72vh'" :empty-state="{
        icon: 'i-heroicons-circle-stack-20-solid', label: 'ไม่มีรายการ.',
      }" :columns="columns" :rows="filteredRows">
        <template v-for="(column, index) in columns" :key="column.key" v-slot:[(getSlotName(column))]="{ row }">
          <span v-if="column.key === 'Material'">
            {{ row[column.key] }}
          </span>

          <span v-else-if="column.key === 'Sum'" class="px-6">
            {{ sum(row) }}
          </span>

          <input v-else v-model="row[column.key]" type="number" :tabindex="index"
            class="min-w-20 bg-gray-50 border focus:outline-none text-sm rounded-lg focus:border-blue-500 block w-full p-2 "
            :class="highlights(row, column.key) ? 'border-green-500' : ''" />
        </template>

      </UTable>
      <div class="flex justify-end px-3 py-3.5">
        <UButton @click="save()" size="lg" color="blue" variant="solid" label="Save" class="px-10" />
      </div>
    </UCard>

    <!-- modal -->
    <UModal v-model="isOpenModal">
      <UCard class="w-full  p-2">
        <div class="w-full h-full flex flex-col space-y-3 justify-center items-center">
          <div class="w-full flex justify-between ">
            <span> Material:</span>
            <UInputMenu :onchange="validateInput()" v-model="addMetInput.meterial" :options="materialData" searchable
              class="w-8/12 rounded-lg"
              :class="(validate.addMetInputMeterial || addMetInput.meterial === '') ? 'border-2 border-red-500 focus:border-red-500' : ''"
              placeholder="Meterial..." inputClass="bg-gray-50  p-2" color="white">
              <template #option-empty="{ query }">
                <q>{{ query }}</q> not found
              </template>
            </UInputMenu>
          </div>
          <div class="w-full flex justify-between">
            <span> Product Code:</span>
            <input v-model="addMetInput.productCode" placeholder="Product Code..."
              class="w-8/12 bg-gray-50 border focus:outline-none text-sm rounded-lg focus:border-blue-500 p-2 " />
          </div>
          <div class="w-full flex justify-start">
            <span> Description:</span>
          </div>
          <textarea v-model="addMetInput.description" :rows="3" placeholder="Description..."
            class="w-full bg-gray-50 border focus:outline-none text-sm rounded-lg focus:border-blue-500 p-2 " />
          <div class="w-full pt-4 flex justify-around">
            <UButton @click="addMaterial" size="lg" color="blue" variant="outline" label="Add" class="px-10" />
            <UButton @click="isOpenModal = false" size="lg" color="red" variant="solid" label="Cancel" class="px-8" />
          </div>
        </div>
      </UCard>
    </UModal>
  </UContainer>
</template>

<script>
import DataJson from "~/assets/data.json";
import MaterialJson from "~/assets/material.json";

export default {
  data() {
    return {
      materialData: [],
      rewDatas: [],
      columns: [],
      rows: [],
      filteredRows: [],
      searchInput: "",
      isOpenModal: false,
      addMetInput: {
        meterial: "",
        productCode: "",
        description: ""
      },
      validate: {
        addMetInputMeterial: false
      }
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      let columnsLocation = [];
      let rowsStore = [];
      let matStore = [];
      if (DataJson) {
        DataJson.forEach((data) => {
          //columns
          if (!columnsLocation.some((e) => e.key === data.Location))
            columnsLocation.push({
              key: data.Location,
              label: data.Location,
            });

          columnsLocation.sort((a, b) =>
            a.key.toLowerCase() > b.key.toLowerCase() ? 1 : -1
          );

          //rows
          const index = rowsStore.findIndex(
            (e) => e.Material === data.Material
          );
          if (index === -1) {
            rowsStore.push({
              Material: data.Material,
              ProductCode: data.ProductCode,
              [data.Location]: data.QTY,
            });
            this.rewDatas.push({
              Material: data.Material,
              ProductCode: data.ProductCode,
              [data.Location]: data.QTY,
            });
          } else {
            rowsStore[index][data.Location] = data.QTY;
            this.rewDatas[index][data.Location] = data.QTY;
          }
        });

        this.columns = [
          { key: "Material", label: "Material", sortable: true },
        ].concat(columnsLocation);
        this.columns.push({ key: "Sum", label: "Sum" });
        this.rows = rowsStore;
      }

      if (MaterialJson) {
        MaterialJson.forEach((mat) => {
          matStore.push(mat.Material);
        })

        this.materialData = matStore;
      }
    },
    getSlotName(column) {
      return `${column.key}-data`;
    },
    sum(rows) {
      let sum = 0;
      for (let a in rows) {
        typeof rows[a] === "number" ? sum += rows[a] : "";
      }
      return sum;
    },
    search() {
      if (this.searchInput === "") {
        this.filteredRows = this.rows;
        return;
      }
      this.filteredRows = this.rows.filter((row) => {
        return String(row.Material).toLowerCase().includes(this.searchInput.toLowerCase())
      })
    },
    highlights(row, columnKey) {
      const i = this.rewDatas.find((rewData) => {
        return rewData.Material === row["Material"];
      })
      if (!i) {
        return row[columnKey] ? true : false;
      }
      return i[columnKey] === row[columnKey] ? false : true;
    },
    validateInput() {
      let val = this.rows.some((row) => {
        return row.Material.toLowerCase() === this.addMetInput.meterial.toLowerCase();
      });
      if (this.addMetInput.meterial === "") {
        val = false;
      }
      val ? this.validate.addMetInputMeterial = true : this.validate.addMetInputMeterial = false;
      return val;
    },
    addMaterial() {
      if (this.validateInput()) {
        return;
      }

      this.rows.push({
        Material: this.addMetInput.meterial,
        ProductCode: this.addMetInput.productCode,
        Description: this.addMetInput.description,
      });
      this.rewDatas.push({
        Material: this.addMetInput.meterial,
        ProductCode: this.addMetInput.productCode,
      });
      this.search();
      this.isOpenModal = false;

      console.log({
        Material: this.addMetInput.meterial,
        ProductCode: this.addMetInput.productCode,
        Description: this.addMetInput.description
      })

      this.addMetInput.meterial = "";
      this.addMetInput.productCode = "";
      this.addMetInput.description = "";
    },
    save() {
      let saveData = [];

      this.rows.forEach((row) => {
        this.rewDatas.forEach((rewdata) => {
          if (row.Material === rewdata.Material) {
            const difference = Object.keys(row).filter(k => row[k] !== rewdata[k]).reduce((obj, key) => {
              return {
                ...obj,
                [key]: row[key]
              };
            }, {});

            if (Object.keys(difference).length > 0) {
              Object.keys(difference).forEach((key) => {
                if (typeof difference[key] === "number") {
                  saveData.push({
                    Material: row.Material,
                    Location: key,
                    QTY: difference[key]
                  });
                }
              });
            }
          }
        })
      })

      console.log(saveData);
    },
  },
};
</script>
