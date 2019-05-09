<template>
  <div>
    <header class="bg-logo-blue px-2">
      <div class="container max-w-lg mx-auto py-2">
        <div class="flex justify-between items-center">
          <img class="h-10" src="../assets/bnl-logo.png" alt="BnL Logo">
          <button @click="toggleModal"
            class="capitalize flex items-center text-blue-lightest font-medium hover:bg-blue-darker px-4 py-2 rounded">
            <svg class="w-6 fill-current mr-1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
              <path fill-rule="evenodd"
                d="M17 11a1 1 0 0 1 0 2h-4v4a1 1 0 0 1-2 0v-4H7a1 1 0 0 1 0-2h4V7a1 1 0 0 1 2 0v4h4z"></path>
            </svg>
            new feed
          </button>
        </div>
      </div>
    </header>
    <main>
      <div class="container max-w-md mx-auto px-2">
        <div v-if="items.length === 0" class="h-full text-center mt-16">
          <p class="font-bold text-3xl mb-5">Get started by adding some RSS feeds</p>
          <div class="flex">
            <div class="w-full">
            </div>
          </div>
        </div>
        <div v-else>
          <h1 class="text-2xl mt-10 mb-5 text text-grey-darkest">Articles</h1>
          <ol class="list-reset">
            <li v-for="item in items" :key="item.key" class="mb-5">
              <a :href="item.link" class="no-underline block bg-white p-4 shadow rounded max-w-sm" target="_blank">
                <h4 class="mb-2 text-black">{{ item.title }}</h4>
                <p class="text-grey-dark text-sm mb-4">{{item.contentSnippet.substring(0, 140)}}...</p>
                <p class="text-grey-darker text-xs">
                  <span v-if="item.creator">
                    {{ item.creator }} &middot;
                  </span>
                  <span v-if="item.pubDate">
                    {{ item.humanDate }}
                  </span>
                </p>
              </a>
            </li>
          </ol>
        </div>
      </div>
    </main>
    <section v-show="toggle" class="flex fixed pin bg-black-25 px-2">
      <div id="modal" class="container max-w-sm m-auto">
        <div class="bg-white px-6 md:px-12 py-10 rounded shadow-lg relative">
          <div class="mb-4">
            <label for="feed-url" class="block font-bold mb-2">
              Add a RSS feed URL
              <span class="text-red">*</span>
            </label>
            <input id="feed-url" name="feed-url" type="text" v-model="feedUrl" autocomplete="off" autofocus
              class="rounded px-3 py-1 leading-loose bg-grey-lighter w-full text-grey-darker text-lg"
              :class="highlight ? 'border border-red-lighter text-red-dark' : ''" placeholder="example-url.com/rss">
          </div>
          <button @click="validateUrl"
            class="rounded w-full bg-logo-blue hover:bg-blue-darker text-blue-lightest font-semibold py-3">Add
            Feed</button>
          <p class="text-grey-dark absolute pin-t pin-r w-8" @click="toggleModal"><svg xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 24 24" class="fill-current">
              <path fill-rule="evenodd"
                d="M15.78 14.36a1 1 0 0 1-1.42 1.42l-2.82-2.83-2.83 2.83a1 1 0 1 1-1.42-1.42l2.83-2.82L7.3 8.7a1 1 0 0 1 1.42-1.42l2.83 2.83 2.82-2.83a1 1 0 0 1 1.42 1.42l-2.83 2.83 2.83 2.82z" />
            </svg></p>

          <p v-show="error"
            class="p-2 mt-4 text-red text-sm bg-red-lightest flex items-center rounded border border-red-lighter">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" class="w-6 fill-current mr-2">
              <path class="text-red-lighter" d="M12 2a10 10 0 1 1 0 20 10 10 0 0 1 0-20z"></path>
              <path class="text-red-light"
                d="M12 18a1.5 1.5 0 1 1 0-3 1.5 1.5 0 0 1 0 3zm1-5.9c-.13 1.2-1.88 1.2-2 0l-.5-5a1 1 0 0 1 1-1.1h1a1 1 0 0 1 1 1.1l-.5 5z">
              </path>
            </svg>
            {{errorMessage}}
          </p>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
  import Parser from "rss-parser";
  import moment from "moment";

  export default {
    data() {
      return {
        toggle: false,
        CORS_PROXY: "https://cors-anywhere.herokuapp.com/",
        feedUrl: "",
        error: false,
        highlight: false,
        errorMessage: "",
        feeds: []
      };
    },
    methods: {
      toggleModal: function () {
        this.toggle = !this.toggle;
        this.resetModal();
      },

      validateUrl: function () {
        let url = this.feedUrl.trim();
        this.resetModal();

        if (url) this.feedUrl = url;
        else {
          this.errorMessage = "Feed URL is required";
          this.error = true;
          this.highlight = true;
        }

        if (this.feeds.filter(feed => feed.feedUrl === this.feedUrl).length == 0) {
          this.fetchFeed();
          this.feedUrl = "";
        } else {
          this.errorMessage = "Feed has already been added";
          this.error = true;
          this.highlight = true;
        }
      },

      fetchFeed: function () {
        let parser = new Parser();

        parser
          .parseURL(this.CORS_PROXY + this.feedUrl)
          .then(feed => {
            this.addFeed(feed);
            this.toggleModal(feed);
          })
          .catch(error => {
            if (error.message == "Status code 404") {
              this.errorMessage = "Invalid URL";
              this.error = true;
            } else {
              this.errorMessage = "Server error, please try again later!";
              this.error = true;
            }
          });
      },

      addFeed: function (feed) {
        this.feeds.push({
          description: feed.description,
          feedUrl: feed.feedUrl,
          language: feed.language,
          lastBuildDate: feed.lastBuildDate,
          link: feed.link,
          title: feed.title,
          items: feed.items
        });
      },

      dateForHumans: function (date) {
        return moment(date).startOf('day').fromNow();
      },

      resetModal: function () {
        this.highlight = false;
        this.error = false;
      }
    },
    computed: {
      items: function () {
        return this.feeds
          .map(feed => feed.items)
          .flat()
          .map((item, index) => (item.key = {
            ...item,
            key: index,
            humanDate: this.dateForHumans(item.pubDate)
          })).sort((a, b) => new Date(b.pubDate) - new Date(a.pubDate));
      }
    }
  };
</script>