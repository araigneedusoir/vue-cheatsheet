<template>
    <div>
        <form @submit.prevent>
                <h3>Create Post</h3>
                <my-input 
                    v-model="post.title"
                    class="input"
                    type="text"
                    placeholder="Title"
                />
                <my-input 
                    v-model="post.body"
                    class="input"
                    type="text"
                    placeholder="Content"
                />
                <my-button 
                    class="btn"
                    @click="createPost"
                >
                    Post
                </my-button>
        </form>
    </div>
</template>

<script>
export default {
    data(){
        return {
            post: {
                title: '',
                body: ''
            }
        }
    },
    methods: {
        createPost() {
            this.post.id = Date.now();
            this.$emit('create', this.post);
            this.post = {
                title: '',
                body: ''
            }   

        }
    }
}
</script>

<style scoped>
form {
    display: flex;
    flex-direction: column;
}

.btn {
    align-self: flex-end;
    margin-top: 15px;
}
</style>