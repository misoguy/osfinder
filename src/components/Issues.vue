<template>
  <v-progress-linear height="4" color="teal" :indeterminate="true" v-if="isFetchingIssues" />
  <v-container grid-list-md v-else>
    <v-layout row wrap>
      <v-flex
        v-if="issues.length < 1"
        xs12 offset-sm2 sm8 offset-md3 md6 offset-xl4 xl4
      >
        <p>
          It seems like there are no issues regarding your watch list.
        </p>
        <v-btn to="/repositories">Add more repositories to watch</v-btn>
      </v-flex>
      <v-flex
        v-else
        xs12 offset-sm2 sm8 offset-md3 md6 offset-xl4 xl4
        v-for="issue in issues"
        :key="issue.id"
      >
        <v-card>
          <v-card-title>
            <v-layout column>
              <h3>
                <a :href="issue.repoUrl" target="_blank" rel="noopener">
                  {{issue.repoNameWithOwner}}
                </a>
              </h3>
              <span>
                <a :href="issue.url" target="_blank" rel="noopener">
                  {{issue.title}}
                </a>
              </span>
              <div class="issue-info">
                <v-icon>access_time</v-icon>
                <span class="created-at">{{new Date(issue.createdAt).toLocaleString()}}</span>
                <v-icon>message</v-icon> 
                <span class="total-comments">{{issue.totalComments}}</span>
              </div>
              <v-layout row wrap>
                <repo-label
                  v-for="label in issue.labels.nodes"
                  :key="label.id"
                  :name="label.name"
                  :color="label.color"
                />
              </v-layout>
            </v-layout>
          </v-card-title>
          <v-card-actions>
            <v-btn v-if="!issue.bodyHTML" flat block>
              NO ISSUE DESCRIPTION
            </v-btn>
            <v-btn v-else-if="issue.bodyHTML && !showDescription[issue.id]" flat block @click="toggleDescription(issue.id)">
              SHOW ISSUE DESCRIPTION 
              <v-icon>keyboard_arrow_down</v-icon>
            </v-btn>
          </v-card-actions>
          <v-slide-y-transition>
            <v-card-text v-show="showDescription[issue.id]">
              <div v-html="issue.bodyHTML" />
              <v-layout row justify-center>
                <v-btn block :href="issue.url" target="_blank" rel="noopener">
                  TO GITHUB ISSUE
                </v-btn>
                <v-btn icon @click="toggleDescription(issue.id)">
                  <v-icon>keyboard_arrow_up</v-icon>
                </v-btn>
              </v-layout>
            </v-card-text>
          </v-slide-y-transition>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script lang="ts">
import Vue from 'vue';
import { mapGetters, mapActions } from 'vuex';
import Label from './Label.vue';

interface IDescription {
  [key: string]: boolean;
}

export default Vue.extend({
  name: 'Issues',
  components: {
    repoLabel: Label,
  },
  created() {
    this.getIssueList(this.watchList);
  },
  data() {
    const showDescription: IDescription = {};
    return {
      showDescription,
    };
  },
  computed: {
    ...mapGetters(['me', 'issues', 'isFetchingIssues', 'watchList']),
  },
  methods: {
    toggleDescription(issueId: string) {
      this.showDescription[issueId] = Vue.set(
        this.showDescription,
        issueId,
        !this.showDescription[issueId]
      );
    },
    ...mapActions(['getIssueList']),
  },
});
</script>

<style scoped>
.progress-linear {
  margin: 0;
}
.issue-info {
  display: flex;
  align-items: center;
  margin: 0.2rem 0 0.5rem;
}
.created-at,
.total-comments {
  margin-left: 0.2rem;
  margin-right: 1rem;
}
</style>
