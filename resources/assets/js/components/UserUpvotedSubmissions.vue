<template>
	<div class="padding-bottom-10 flex1" :class="{'flex-center' : nothingFound}" id="submissions" 
		v-infinite-scroll="loadMore" infinite-scroll-disabled="cantLoadMore" @scroll.passive="scrolled"
	>
		<div v-for="submission in submissions" v-bind:key="submission.id">
			<submission :list="submission"></submission>
		</div>
	
		<no-content v-if="nothingFound" :text="'This user has not upvoted any submissions yet'"></no-content>
	
		<loading v-if="loading && page > 1"></loading>
	
		<no-more-items :text="'No more items to load'" v-if="NoMoreItems && !nothingFound"></no-more-items>
	</div>
</template>

<script>
import Submission from '../components/Submission.vue';
import NoContent from '../components/NoContent.vue';
import Loading from '../components/Loading.vue';
import NoMoreItems from '../components/NoMoreItems.vue';
import Helpers from '../mixins/Helpers';

export default {
    mixins: [Helpers],

    components: {
        Submission,
        Loading,
        NoContent,
        NoMoreItems
    },

    computed: {
        NoMoreItems() {
            return Store.page.user.upVotedSubmissions.NoMoreItems;
        },

        submissions() {
            return Store.page.user.upVotedSubmissions.submissions;
        },

        loading() {
            return Store.page.user.upVotedSubmissions.loading;
        },

        page() {
            return Store.page.user.upVotedSubmissions.page;
        },

        nothingFound() {
            return Store.page.user.upVotedSubmissions.nothingFound;
        },

        cantLoadMore() {
            return this.loading || this.NoMoreItems || this.nothingFound;
        }
    },

    beforeRouteEnter(to, from, next) {
        if (
            typeof Store.page.user.temp.username != 'undefined' &&
            Store.page.user.temp.username != to.params.username
        ) {
            Store.page.user.upVotedSubmissions.clear();
        }

        if (Store.page.user.upVotedSubmissions.page === 0) {
            if (typeof app != 'undefined') {
                app.$Progress.start();
            }

            Promise.all([
                Store.page.user.upVotedSubmissions.getSubmissions(),
                Store.page.user.getUser(to.params.username)
            ]).then(() => {
                next((vm) => {
                    vm.$Progress.finish();
                });
            });
        } else {
            next((vm) => {
                vm.$Progress.finish();
            });
        }
    },

    beforeRouteUpdate(to, from, next) {
        if (to.hash !== from.hash) return;

        Store.page.user.upVotedSubmissions.clear();

        this.$Progress.start();

        Promise.all([
            Store.page.user.upVotedSubmissions.getSubmissions(),
            Store.page.user.getUser(to.params.username, false)
        ]).then((values) => {
            Store.page.user.setUser(values[1]);
            this.setPageTitle('Up-voted Submissions | @' + to.params.username);
            this.$Progress.finish();
            next();
        });
    },

    methods: {
        loadMore() {
            Store.page.user.upVotedSubmissions.getSubmissions();
        }
    }
};
</script>
