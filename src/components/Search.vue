<template>
  <div>
    <v-layout row align-center>
      <v-flex xs5 md6>
        <v-select
          v-bind:items="languages"
          v-model="filters.language"
          label="Language"
          autocomplete
        />
      </v-flex>
      <v-flex xs5 md6>
        <v-text-field
          v-model="searchText"
          label="Repository"
          @keyup.native="onKeyUp"
        />
      </v-flex>
      <v-spacer />
      <v-flex>
        <v-btn icon dark color="green" @click="searchRepos">
          <v-icon>search</v-icon>
        </v-btn>
      </v-flex>
    </v-layout>
    <v-layout v-if="isLoading" justify-center>
      <v-progress-circular indeterminate color="primary" />
    </v-layout>
    <v-layout v-else column>
      <repo-card
        v-for="repo in repositories.nodes"
        :key="repo.url"
        :initial-repo="repo"
      />
      <infinite-loading
        v-if="repositories.nodes.length > 0"
        @infinite="infiniteHandler"
        spinner="waveDots"
      >
        <span slot="no-more">
          No more repository
        </span>
      </infinite-loading>
    </v-layout>
  </div>
</template>


<script lang="ts">
import Vue from 'vue';
import InfiniteLoading from 'vue-infinite-loading';
import SearchRepositories from '@/graphql/SearchRepositories';
import githubQueryBuilder from '@/graphql/githubQueryBuilder';
import githubLanguages from '@/graphql/githubLanguages';
import client from '@/apolloClient';

import RepositoryCard from './RepositoryCard.vue';

interface IData {
  isLoading: boolean;
  repoName: string;
  searchText: string;
  languages: any[];
  filters: any;
  qualifiers: any;
  repositories: any;
}

export default Vue.extend({
  name: 'Search',
  components: {
    repoCard: RepositoryCard,
    InfiniteLoading,
  },
  mounted() {
    this.searchRepos();
  },
  data(): IData {
    return {
      isLoading: true,
      repoName: '',
      searchText: '',
      languages: githubLanguages,
      filters: {
        language: 'JavaScript',
      },
      qualifiers: {
        language: 'JavaScript',
      },
      repositories: {
        nodes: [],
        pageInfo: {},
      },
    };
  },
  computed: {
    q(): string {
      return githubQueryBuilder(this.repoName, this.qualifiers);
    },
  },
  methods: {
    onKeyUp(e: KeyboardEvent) {
      const ENTER = 13;
      if (e.keyCode === ENTER) {
        this.searchRepos();
      }
    },
    searchRepos() {
      this.repoName = this.searchText;
      this.qualifiers = this.filters;
      this.isLoading = true;
      client
        .query({
          query: SearchRepositories,
          variables: {
            q: this.q,
          },
        })
        .then(({ data }: { data: any }) => {
          this.repositories.nodes = data.search.nodes;
          this.repositories.pageInfo = data.search.pageInfo;
          this.isLoading = false;
        });
    },
    infiniteHandler($state: any) {
      if (this.repositories.pageInfo.hasNextPage) {
        client
          .query({
            query: SearchRepositories,
            variables: {
              q: this.q,
              after: this.repositories.pageInfo.endCursor,
            },
          })
          .then(({ data }: { data: any }) => {
            this.repositories.nodes = this.repositories.nodes.concat(
              data.search.nodes
            );
            this.repositories.pageInfo = data.search.pageInfo;
            const hasNextPage = data.search.pageInfo.hasNextPage;
            $state.loaded();
            if (!hasNextPage) {
              $state.complete();
            }
          });
      } else {
        $state.loaded();
        $state.complete();
      }
    },
  },
});
</script>
