<template>
  <Generic :item="item" class="webhook">
    <template #content>
      <div class="is-flex is-align-items-center">
        <p class="title is-4">
          {{ item.name }}
        </p>
        <div v-if="isTypeService" class="ml-auto status" :class="status">
          {{ status }}
        </div>
      </div>

      <p class="subtitle is-6 pt-3">
        <template v-if="isTypeText">
          <pre
            v-if="content"
            class="has-text-inherit has-background-inherit"
          ><code>{{ content }}</code></pre>
        </template>
        <template v-else-if="isTypeService">
          <button
            class="button is-fullwidth has-text-weight-bold"
            :class="[isActiveService ? 'is-danger' : 'is-success']"
            :disabled="loading || isDeadService"
            @click="toggleActiveState"
          >
            {{ isActiveService ? "Stop" : "Start" }}
          </button>
        </template>
      </p>
    </template>
  </Generic>
</template>

<script>
import service from "@/mixins/service.js";
import Generic from "./Generic.vue";

export default {
  name: "Webhook",
  components: {
    Generic,
  },
  mixins: [service],
  props: {
    item: Object,
  },
  data: () => ({
    content: "",
    status: "dead",
    loading: false,
  }),
  computed: {
    isTypeText() {
      return this.item.subtype === "text";
    },
    isTypeService() {
      return this.item.subtype === "service";
    },
    isActiveService() {
      return this.status === "running";
    },
    isDeadService() {
      return this.status === "dead";
    },
    headers: function () {
      return {
        ...(this.item.apikey
          ? { Authorization: `Bearer ${this.item.apikey}` }
          : {}),
      };
    },
  },
  created() {
    if (this.isTypeText) {
      this.fetchTextContent();
    } else if (this.isTypeService) {
      this.fetchActiveStateContent();
    } else {
      console.log("wrong subtype");
    }
  },
  methods: {
    fetchTextContent: async function () {
      const headers = this.headers;
      this.loading = true;

      return this.fetch(this.item.actions, { headers }, false)
        .then((response) => {
          this.content = response;
        })
        .finally(() => {
          this.loading = false;
        });
    },
    fetchActiveStateContent: async function () {
      const headers = this.headers;
      this.loading = true;

      return this.fetch(this.item.actions.active, { headers })
        .then((response) => {
          this.status = response?.active ? "running" : "stopped";
        })
        .catch((e) => {
          console.log(e);
          this.status = "dead";
        })
        .finally(() => {
          this.loading = false;
        });
    },
    toggleActiveState() {
      const headers = this.headers;
      const action = this.isActiveService
        ? this.item.actions.stop
        : this.item.actions.start;
      this.loading = true;

      return this.fetch(action, { headers, method: "POST" })
        .then((response) => {
          this.status = response?.active ? "running" : "stopped";
        })
        .catch((e) => {
          console.log(e);
          this.status = "dead";
        })
        .finally(() => {
          this.loading = false;
        });
    },
  },
};
</script>

<style lang="scss">
.webhook {
  max-width: 100%;

  .card-content {
    height: auto !important;
  }
}
</style>
<style scoped lang="scss">
.status {
  font-size: 0.8rem;
  color: var(--text-title);

  &.running:before {
    background-color: #94e185;
    border-color: #78d965;
    box-shadow: 0 0 5px 1px #94e185;
  }

  &.stopped:before {
    background-color: #c9404d;
    border-color: #c42c3b;
    box-shadow: 0 0 5px 1px #c9404d;
  }

  &:before {
    content: " ";
    display: inline-block;
    width: 7px;
    height: 7px;
    margin-right: 10px;
    border: 1px solid #000;
    border-radius: 7px;
  }
}
</style>
