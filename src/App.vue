<template>
    <div class="app">
        <div class="app-btns">
            <my-input 
                v-model="searchQuerry"
                placeholder="Search"
            >
            </my-input>
            <my-button
                @click="showDialog"
            >
                Create Post
            </my-button>
            <my-select 
                v-model="selectedSortMethod"
                :options="sortingOptions"
            />
        </div>

        <my-dialog v-model:show="dialogVisibility">
            <post-form 
                @create="createPost"
            />
        </my-dialog>

        <post-list
            :posts="sortedAndSearchedPosts"
            @remove="removePost"
            v-if="!postsAreLoading"
        />
        <div v-else>Loading posts...</div>
        <div ref="observer" class="observer"></div>
        <!-- <div class="page-wrapper">
            <div
                v-for="pageNumber in totalPages"
                :key="pageNumber"
                class="page"
                :class="{
                    'current-page': pageNumber == page 
                }"
                @click="changePage(pageNumber)"
            >{{ pageNumber }}</div>
        </div> -->
    </div>
</template>

<script>
import axios from 'axios'
import PostForm from '@/components/PostForm.vue'
import PostList from '@/components/PostList.vue'

export default {
    components: {
        PostList, PostForm
    },
    data() {
        return {
            posts: [],
            page: 1,
            limit: 10,
            totalPages: 0,
            dialogVisibility: false,
            postsAreLoading: false,
            selectedSortMethod: '',
            searchQuerry: '',
            sortingOptions: [
                {
                    value: 'title',
                    name: 'Sort by title'
                },
                {
                    value: 'body',
                    name: 'Sort by content'
                },
                {
                    value: 'id',
                    name: 'Sort by ID'
                }
            ]
        }
    },
    methods: {
        createPost(post){
            this.posts.push(post)
            this.dialogVisibility = false
        },
        removePost(post){
            this.posts = this.posts.filter (p => p.id !== post.id)
        },
        showDialog() {
            this.dialogVisibility = true
        },
        // changePage(pageNumber) {
        //     this.page = pageNumber
        // },
        async fetchPosts() {
            try {
                this.postsAreLoading = true
                const response = await axios.get('https://jsonplaceholder.typicode.com/posts', {
                    params: {
                        _page: this.page,
                        _limit: this.limit
                    }
                })
                this.totalPages = Math.ceil(response.headers['x-total-count'] / this.limit)
                this.posts = response.data
            } catch (e) {
                alert('Error')
            } finally {
                this.postsAreLoading = false
            }
        },
        async fetchMorePosts() {
            try {
                this.page += 1
                const response = await axios.get('https://jsonplaceholder.typicode.com/posts', {
                    params: {
                        _page: this.page,
                        _limit: this.limit
                    }
                })
                this.totalPages = Math.ceil(response.headers['x-total-count'] / this.limit)
                this.posts = [...this.posts, ...response.data]
            } catch (e) {
                alert('Error')
            }
        }
    },
    mounted() {
        this.fetchPosts()
        const options = {
            rootMargin: '8px',
            threshold: 1.0
        }
        const callback = (entries, observer) => {
            if (entries[0].isIntersecting && this.totalPages > this.page) {
                this.fetchMorePosts()
            }
        }
        const observer = new IntersectionObserver(callback, options)
        observer.observe(this.$refs.observer)
    },
    computed: {
        sortedPosts() {
            if (this.selectedSortMethod == 'title' || this.selectedSortMethod == 'body') {
                return [...this.posts].sort((a, b) => a[this.selectedSortMethod]?.localeCompare(b[this.selectedSortMethod]))
            }
            else {
                return [...this.posts]
            }
        },
        sortedAndSearchedPosts() {
            return this.sortedPosts.filter(post => post.title.toLowerCase().includes(this.searchQuerry.toLowerCase()))
        }
    },
    watch: {
        // page() {
        //     this.fetchPosts()
        // }
    }
}
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

.app {
    padding: 20px;
}

.app-btns {
    display: flex;
    justify-content: space-between;
}

/* .page-wrapper {
    display: flex;
    margin-top: 15px;
}

.page {
    border: 1px solid black;
    padding: 10px;
}

.current-page {
    border: 2px solid teal;

} */

.observer {
    height: 30px;
    background: teal;
}
</style>