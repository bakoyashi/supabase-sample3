<template>
  <v-container>
    <h1>Data from Supabase</h1>
    <v-data-table
      :headers="headers"
      :items="books"
      class="elevation-1"
    >
      <template v-slot:item.actions="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)">
          mdi-pencil
        </v-icon>
        <v-icon small @click="deleteItem(item)">
          mdi-delete
        </v-icon>
      </template>
    </v-data-table>
    <v-dialog v-model="dialog" max-width="500px">
      <template v-slot:activator="{ on, attrs }">
        <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">New Item</v-btn>
      </template>
      <v-card>
        <v-card-title>
          <span class="headline">{{ editedIndex === -1 ? 'New Item' : 'Edit Item' }}</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12" sm="6" md="4">
                <v-text-field v-model="editedItem.title" label="Title"></v-text-field>
              </v-col>
              <v-col cols="12" sm="6" md="4">
                <v-text-field v-model="editedItem.read_date" label="Read Date"></v-text-field>
              </v-col>
              <v-col cols="12" sm="6" md="4">
                <v-text-field v-model="editedItem.rating" label="Rating"></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field v-model="editedItem.review" label="Review"></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
          <v-btn color="blue darken-1" text @click="save">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import { ref, onMounted } from 'vue';
import { createClient } from '@supabase/supabase-js';

export default {
  name: 'HelloWrold',
  setup() {
    const SUPABASE_URL = process.env.VUE_APP_SUPABASE_URL;
    const SUPABASE_ANON_KEY = process.env.VUE_APP_SUPABASE_ANON_KEY;

    const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

    const headers = [
      { text: 'Title', value: 'title' },
      { text: 'Read Date', value: 'read_date' },
      { text: 'Rating', value: 'rating' },
      { text: 'Review', value: 'review' },
      { text: 'Actions', value: 'actions', sortable: false }
    ];

    const books = ref([]);
    const dialog = ref(false);
    const editedIndex = ref(-1);
    const editedItem = ref({
      title: '',
      read_date: '',
      rating: '',
      review: ''
    });

    onMounted(async () => {
      let { data, error } = await supabase.from('books').select('*');
      if (error) {
        console.error('Error: ', error);
      } else {
        books.value = data;
      }
    });

    const close = () => {
      dialog.value = false
      setTimeout(() => {
        editedItem.value = Object.assign({}, defaultItem)
        editedIndex.value = -1
      }, 300)
    }

    const save = async () => {
      if (editedIndex.value > -1) {
        let { data, error } = await supabase.from('books').update(editedItem.value).match({ id: editedItem.value.id });
        if (error) {
          console.error('Error: ', error);
        } else {
          Object.assign(books[editedIndex.value], editedItem.value)
        }
      } else {
        let { data, error } = await supabase.from('books').insert([editedItem.value]);
        if (error) {
          console.error('Error: ', error);
        } else {
          books.value.push(data[0]);
        }
      }
      close()
    }

    const deleteItem = async (item) => {
      let { data, error } = await supabase.from('books').delete().match({ id: item.id });
      if (error) {
        console.error('Error: ', error);
      } else {
        books.value.splice(books.value.indexOf(item), 1)
      }
    }

    const editItem = (item) => {
      editedIndex.value = books.value.indexOf(item)
      editedItem.value = Object.assign({}, item)
      dialog.value = true
    }

    return { headers, books, dialog, close, save, deleteItem, editItem };
  },
};
</script>

<!-- <template>
  <div>
    <h1>Data from Supabase</h1>
    <ul>
      <li v-for="book in books" :key="book.id">
        Title: {{ book.title }}<br/>
        Read Date: {{ book.read_date }}<br/>
        Rating: {{ book.rating }}<br/>
        Review: {{ book.review }}
      </li>
    </ul>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import { createClient } from '@supabase/supabase-js';

export default {
  name: 'HelloWrold',
  setup() {
    const SUPABASE_URL = process.env.VUE_APP_SUPABASE_URL;
    const SUPABASE_ANON_KEY = process.env.VUE_APP_SUPABASE_ANON_KEY;
    console.log(SUPABASE_URL)
    console.log(SUPABASE_ANON_KEY)

    const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

    const books = ref([]);

    onMounted(async () => {
      let { data, error } = await supabase.from('books').select('*');
      if (error) {
        console.error('Error: ', error);
      } else {
        books.value = data;
      }
    });

    return { books };
  },
};
</script>


<!-- <template>
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

      <v-col class="mb-4">
        <h1 class="display-2 font-weight-bold mb-3">
          Hello to the Vuetify 3 Beta
        </h1>


        <p class="subheading font-weight-regular">
          For help and collaboration with other Vuetify developers,
          <br>please join our online
          <a
            href="https://community.vuetifyjs.com"
            target="_blank"
          >Discord Community</a>
        </p>
      </v-col>

      <v-col
        class="mb-5"
        cols="12"
      >
        <h2 class="headline font-weight-bold mb-5">
          What's next?
        </h2>

        <v-row justify="center">
          <a
            v-for="(next, i) in whatsNext"
            :key="i"
            :href="next.href"
            class="subheading mx-3"
            target="_blank"
          >
            {{ next.text }}
          </a>
        </v-row>
      </v-col>

      <v-col
        class="mb-5"
        cols="12"
      >
        <h2 class="headline font-weight-bold mb-5">
          Important Links
        </h2>

        <v-row justify="center">
          <a
            v-for="(link, i) in importantLinks"
            :key="i"
            :href="link.href"
            class="subheading mx-3"
            target="_blank"
          >
            {{ link.text }}
          </a>
        </v-row>
      </v-col>

      <v-col
        class="mb-5"
        cols="12"
      >
        <h2 class="headline font-weight-bold mb-5">
          Ecosystem
        </h2>

        <v-row justify="center">
          <a
            v-for="(eco, i) in ecosystem"
            :key="i"
            :href="eco.href"
            class="subheading mx-3"
            target="_blank"
          >
            {{ eco.text }}
          </a>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>

export default {
  name: 'HelloWorld',

  data: () => ({
    ecosystem: [
      {
        text: 'vuetify-loader',
        href: 'https://github.com/vuetifyjs/vuetify-loader/tree/next',
      },
      {
        text: 'github',
        href: 'https://github.com/vuetifyjs/vuetify/tree/next',
      },
      {
        text: 'awesome-vuetify',
        href: 'https://github.com/vuetifyjs/awesome-vuetify',
      },
    ],
    importantLinks: [
      {
        text: 'Chat',
        href: 'https://community.vuetifyjs.com',
      },
      {
        text: 'Made with Vuetify',
        href: 'https://madewithvuejs.com/vuetify',
      },
      {
        text: 'Twitter',
        href: 'https://twitter.com/vuetifyjs',
      },
      {
        text: 'Articles',
        href: 'https://medium.com/vuetify',
      },
    ],
    whatsNext: [
      {
        text: 'Explore components',
        href: 'https://vuetifyjs.com',
      },
      {
        text: 'Roadmap',
        href: 'https://vuetifyjs.com/introduction/roadmap/',
      },
      {
        text: 'Frequently Asked Questions',
        href: 'https://vuetifyjs.com/getting-started/frequently-asked-questions',
      },
    ],
  }),
}
</script> --> -->
