<template>
    <section class="posts-container">
        <breadcrumb :pages="breadcrumbsData"/>
        <div class="mobile-container">
            <div class="post-heading-filters">
                <div class="post-heading">
                    <div class="user-avatar">
                        <img src="@/assets/images/icons-category.svg" alt="LV">
                    </div>
                    <h1>Posts</h1>
                </div>
                <filters route-name="posts"></filters>
            </div>
            <div class="article-grid">
                <template v-for="(item, key) in posts.data">
                    <article-item :title="item.title"
                                  :image="item.image_url"
                                  :description="item.excerpt"
                                  :key="key"
                                  :to="{ name: 'slug', params: { slug: item.slug }}"
                    />
                </template>
            </div>

            <div class="text-center">
              <button @click="showMore()" class="button"
                    v-if="hasMorePages && !show_more">
                {{ $apollo.queries.posts.loading ? 'Loading ...' : 'Show more'}}
              </button>
            </div>

            <no-ssr>
                <infinite-loading @infinite="showMore"
                                  v-if="show_more"></infinite-loading>
            </no-ssr>
        </div>
    </section>
</template>

<script>
    import postsQql from '@/graphql/queries/post/all.graphql';

    export default {
        components: {
            filters: () => import('@/components/mobile/elements/filters'),
            Breadcrumb: () => import('@/components/shared/partials/elements/breadcrumb'),
            ArticleItem: () => import('@/components/shared/partials/elements/article-item')
        },
        name: "posts",
        apollo: {
            posts: {
                query: postsQql,
                variables() {
                    // Initial variables
                    return {
                        count: this.$route.query.count ?? 12,
                        sort_by: this.$route.query.sort_by ?? 'latest',
                        s: this.$route.query.s
                    }
                }
            },
        },
        computed: {
            hasMorePages() {
                return this.posts.paginatorInfo && this.posts.paginatorInfo.hasMorePages;
            }
        },
        methods: {
            showMore($state) {
                if (!this.hasMorePages || this.$apollo.queries.posts.loading) {
                    return true
                }

                this.page++;

                this.$apollo.queries.posts.fetchMore({
                    // New variables
                    variables: {
                        count: this.$route.query.count ?? 12,
                        sort_by: this.$route.query.sort_by ?? 'latest',
                        s: this.$route.query.s,
                        page: this.page
                    },
                    // Transform the previous result with new data
                    updateQuery: (previousResult, {fetchMoreResult}) => {
                        if (!fetchMoreResult)
                            return previousResult;

                        if ($state) {
                            if (fetchMoreResult.posts.paginatorInfo.hasMorePages)
                                $state.loaded();
                            else
                                $state.complete();
                        }

                        this.show_more = true;
                        fetchMoreResult.posts.data = [...previousResult.posts.data, ...fetchMoreResult.posts.data];
                        return fetchMoreResult;
                    },
                })
            },
        },
        mounted() {
        },
        data() {
            return {
                posts: {},
                show_more: false,
                page: 1,
                breadcrumbsData: [
                    {
                        name: 'Home',
                        link: "/"
                    },
                    {
                        name: 'Posts',
                        link: "/posts"
                    }
                ]
            }
        }
    }

</script>

<style lang="stylus" scoped>
    .posts-container
        padding-bottom 80px

    .post-heading-filters
        position relative
        margin 20px 0 30px

    .post-heading
        text-align center

        h1
            font-size 28px
            font-weight 600
            color $tertiary
            margin-top 10px
            line-height 1

    .article-grid
        display grid
        grid-template-columns repeat(1, 1fr)
        grid-gap 40px 10px

    .ads
        width 300px
        height 250px
        display flex
        align-items center
        justify-content center
        background-color transparent
        font-size 24px
        color $white
        margin 60px auto

    .adsbygoogle
        margin 15px 0

  .text-center
    padding-top 30px

</style>
