<template>
  <div class="blog">
    <nav-bar active_page="Blog"></nav-bar>
    <v-content>
      <v-app-bar v-if="show_app_bar()" color="#fff" light flat>
       <v-toolbar-title>Logged in as admin</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn @click="show_add_blog = !show_add_blog" icon>
          <v-icon v-if="!show_add_blog">fa-plus</v-icon>
          <v-icon v-else>fa-close</v-icon>
        </v-btn>
        <v-btn href="/logout" color="dark" dark>Logout</v-btn>
      </v-app-bar>
      <add-external-blog-links v-if="show_add_blog" @submitted="get_bogs" style="margin: auto">
      </add-external-blog-links>
      <v-container :style="main_content_css">
        <v-progress-linear :active="loading" :indeterminate="loading"
                           absolute top color="black accent-4">
        </v-progress-linear>
        <div class="pa-1" v-for="blog in blogs" :key="blog.title">
          <v-divider></v-divider>
          <regular-old-card :title="blog.title" :description="blog.description"
                            :url="blog.url" :image_url="blog.image_url"></regular-old-card>
        </div>
      </v-container>
    </v-content>
    <div v-if="this.is_mobile">
      <footer-comp></footer-comp>
    </div>
  </div>
</template>

<script>
import AddExternalBlogLinks from '../components/AddExternalBlogLinks.vue';
import NavBar from '../components/NavBar.vue';
import Footer from '../components/Footer.vue';
import RegularOldCard from '../components/RegularOldCard.vue';
import mobile from '../js/utils';

export default {
  name: 'Blog',
  data() {
    return {
      page_name: 'Blog',
      blogs: [],
      updated: '',
      loading: true,
      is_admin: false,
      show_add_blog: false,
      is_mobile: false,
    };
  },
  methods: {
    get_bogs() {
      // Get all public repos
      this.$http.get(`${this.$backend_address}/blogs`)
        .then((response) => {
          this.loading = false;
          this.blogs = response.data.blogs;
          this.updated = response.data.updated;
        });
    },
    show_app_bar() {
      if (mobile.isMobile()) {
        return false;
      }
      return this.is_admin;
    },
  },
  computed: {
    main_content_css() {
      if (this.is_mobile) {
        return {
          paddingRight: '0rem',
          paddingLeft: '0rem',
        };
      }
      return {
        paddingRight: '2rem',
        paddingLeft: '2rem',
      };
    },
  },
  components: {
    navBar: NavBar,
    regularOldCard: RegularOldCard,
    addExternalBlogLinks: AddExternalBlogLinks,
    footerComp: Footer,
  },
  created() {
    this.get_bogs();
    if (mobile.isMobile()) {
      this.is_mobile = true;
    }
    this.$is_authenticated()
      .then((response) => {
        if (response.data.is_authenticated) {
          this.is_admin = true;
        }
      });
  },
};
</script>
