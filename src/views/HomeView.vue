<template>
  <v-row class="mt-10">
    <v-col cols="12" xl="4" lg="4" md="4" sm="12" xs="12" class="mx-auto">
      <v-card min-height="70vh" class="pa-2"
        ><v-overlay :absolute="true" v-model="overlay">
          <div class="lds-facebook">
            <div></div>
            <div></div>
            <div></div></div
        ></v-overlay>
        <v-form class="w-100" @submit.prevent="submitData()">
          <v-row>
            <!-- main category select list -->
            <v-col
              cols="12"
              xl="12"
              lg="12"
              md="12"
              sm="12"
              xs="12"
              class="mx-auto"
            >
              <v-autocomplete
                outlined
                dense
                label="Main Category"
                :items="main_categories"
                v-model="main_category"
              ></v-autocomplete>
            </v-col>

            <!-- sub category select list -->
            <v-col
              cols="12"
              xl="12"
              lg="12"
              md="12"
              sm="12"
              xs="12"
              class="mx-auto"
            >
              <v-autocomplete
                outlined
                dense
                label="Sub Category"
                :items="sub_categories"
                v-model="sub_category"
              ></v-autocomplete>
            </v-col>

            <!-- searchable content -->
            <v-col
              cols="12"
              xl="12"
              lg="12"
              md="12"
              sm="12"
              xs="12"
              class="mx-auto"
              v-for="(option, i) in sub_children"
              :key="i"
            >
              <v-autocomplete
                outlined
                dense
                :label="option.title"
                :items="option.filtered_options"
                v-model="editedItem[option.title]"
                @input="getChildren(option, editedItem[option.title])"
              ></v-autocomplete>

              <v-text-field
                class="mt-2"
                v-if="editedItem[option.title] == 'other'"
                outlined
                dense
                label="please enter the new value ..."
                v-model="editedItem[option.title + 1]"
              ></v-text-field>
              <v-autocomplete
                v-for="(child, x) in option.children"
                :key="x"
                outlined
                dense
                :label="child.title"
                :items="child.options"
                v-model="editedItem[option.title + child.title]"
                @input="
                  getChildOfChild(
                    option,
                    child,
                    editedItem[option.title + child.title]
                  )
                "
              ></v-autocomplete>
              <v-autocomplete
                v-for="(child, x) in option.sub_children"
                :key="x"
                outlined
                dense
                :label="child.title"
                :items="child.options"
                v-model="editedItem[option.title + child.title]"
              ></v-autocomplete>
            </v-col>
            <v-col
              cols="12"
              xl="12"
              lg="12"
              md="12"
              sm="12"
              xs="12"
              class="mx-auto"
            >
              <v-btn class="w-100" color="primary" dark type="submit"
                >submit</v-btn
              >
            </v-col>
          </v-row>
        </v-form></v-card
      >
    </v-col>
    <v-dialog
      v-model="show_submit"
      :width="
        $vuetify.breakpoint.name == 'xs' || $vuetify.breakpoint.name == 'sm'
          ? '98vw'
          : '45vw'
      "
      scrollable
    >
      <v-card class="pa-2">
        <v-card-text style="min-height: 50rem">
          <v-data-table
            :headers="headers"
            :items="submit_items"
            :items-per-page="20"
            class="elevation-1 mt-5"
          ></v-data-table>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      all_categories: [],
      main_categories: [],
      main_category: "",
      sub_categories: [],
      sub_category: "",
      sub_children: [],
      editedItem: {},
      children: [],
      overlay: false,
      show_submit: false,
      headers: [
        {
          text: "Key",
          align: "start",
          sortable: false,
          value: "name",
        },
        {
          text: "value",
          align: "start",
          sortable: true,
          value: "value",
        },
      ],
      submit_items: [],
    };
  },
  watch: {
    main_category(val) {
      if (val) {
        this.get_sub_categories(val);
      }
    },
    sub_category(val) {
      if (val) {
        this.get_searchable_menu(val);
      }
    },
  },
  methods: {
    //method to get the main categories of the web site
    get_categories() {
      this.$axios
        .get("get_all_cats")
        .then((res) => {
          //assign all categories to differnet array from the main category selection
          this.all_categories = res.data.data.categories;
          this.main_categories = res.data.data.categories.map((category) => {
            return {
              text: category.name,
              value: category.id,
            };
          });
        })
        .catch((err) => {
          this.overlay = false;
        });
    },

    //method to get the sub categories of the web site
    get_sub_categories(id) {
      this.overlay = true;
      //filter through the categories to fetch the children of the main category
      let category = this.all_categories.filter((cat) => {
        return cat.id == id;
      });

      this.sub_categories = category[0].children.map((item) => {
        return {
          text: item.name,
          value: item.id,
        };
      });
      this.overlay = false;
    },
    get_searchable_menu(id) {
      this.overlay = true;
      this.$axios
        .get(`properties?cat=${id}`)
        .then((res) => {
          this.sub_children = res.data.data.map((item) => {
            return {
              id: item.id,
              title: item.name,
              filtered_options: item.options
                .map((opt) => {
                  return {
                    text: opt.name,
                    value: opt.id,
                  };
                })
                .concat([
                  {
                    text: "other",
                    value: "other",
                  },
                ]),
            };
          });
          this.overlay = false;
        })
        .catch((err) => {
          this.overlay = false;
        });
    },
    getChildren(option, id) {
      this.overlay = true;
      let index = this.sub_children.indexOf(option);

      this.$axios
        .get(`get-options-child/${id}`)
        .then((res) => {
          this.children = res.data.data.map((child) => {
            return {
              id: child.id,
              title: child.name,
              options: child.options.map((opt) => {
                return {
                  text: opt.name,
                  value: opt.id,
                };
              }),
            };
          });
          this.sub_children[index].children = this.children;
          this.overlay = false;
        })
        .catch((err) => {
          this.overlay = false;
        });
    },
    getChildOfChild(option, child, id) {
      this.overlay = true;
      let index = this.sub_children.indexOf(option);

      this.$axios
        .get(`get-options-child/${id}`)
        .then((res) => {
          this.children = res.data.data.map((child) => {
            return {
              id: child.id,
              title: child.name,
              options: child.options.map((opt) => {
                return {
                  text: opt.name,
                  value: opt.value,
                };
              }),
            };
          });
          this.sub_children[index].sub_children = this.children;
          this.overlay = false;
        })
        .catch((err) => {
          this.overlay = false;
        });
    },
    submitData() {
      Object.entries(this.editedItem).forEach((entry) => {
        this.submit_items.push({
          name: entry[0],
          value: entry[1],
        });
      });
      this.show_submit = true;
    },
  },
  created() {
    // we fetch the initilization data here before dom is created and mounted
    this.get_categories();
  },
};
</script>
<style lang="scss">
.w-100 {
  width: 100% !important;
}
.lds-facebook {
  display: inline-block;
  position: relative;
  width: 80px;
  height: 80px;
}
.lds-facebook div {
  display: inline-block;
  position: absolute;
  left: 8px;
  width: 16px;
  background: #fff;
  animation: lds-facebook 1.2s cubic-bezier(0, 0.5, 0.5, 1) infinite;
}
.lds-facebook div:nth-child(1) {
  left: 8px;
  animation-delay: -0.24s;
}
.lds-facebook div:nth-child(2) {
  left: 32px;
  animation-delay: -0.12s;
}
.lds-facebook div:nth-child(3) {
  left: 56px;
  animation-delay: 0;
}
@keyframes lds-facebook {
  0% {
    top: 8px;
    height: 64px;
  }
  50%,
  100% {
    top: 24px;
    height: 32px;
  }
}
</style>