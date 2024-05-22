<template>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Post Management</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">

</head>

<body>
    <nav class="navbar navbar-expand-lg navbar-light">
        <div class="container d-flex justify-content-end">
            <ul class="nav">
                <li class="nav-item">
                    <router-link to="/logout" class="nav-link">Logout</router-link>
                </li>
            </ul>
        </div>
    </nav>

    <div class="container mt-5">
        <h2>Posts</h2>
        <div v-for="post in posts" :key="post.id" class="card">
            <div class="card-body">
                <h5 class="card-title">{{ post.title }}</h5>
                <p class="card-text">{{ post.content }}</p>
                <p class="card-text"><small class="text-muted">by {{ post.user.name }}</small></p>
                <button class="btn btn-primary" @click="editPost(post)">Edit</button>
                <button class="btn btn-danger" @click="deletePost(post.id)">Delete</button>

                <div class="mt-3">
                    <h6>Comments</h6>
                    <div v-for="comment in post.comments" :key="comment.id" class="mb-2">
                        <p>{{ comment.content }} - <small>{{ comment.user.name }}</small></p>
                        <button class="btn btn-sm btn-primary" @click="editComment(comment)">Edit</button>
                        <button class="btn btn-sm btn-danger" @click="deleteComment(comment.id)">Delete</button>
                    </div>
                    <form @submit.prevent="addComment(post.id)">
                        <div class="input-group">
                            <input type="text" v-model="newComment" placeholder="Add a comment" class="form-control">
                            <button type="submit" class="btn btn-success">Post</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
        <button class="btn btn-success" @click="createPost">Add New Post</button>

        <!-- Modal for creating/updating post -->
        <div v-if="showPostModal" class="modal show d-block" tabindex="-1">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">{{ editingPost ? 'Edit Post' : 'New Post' }}</h5>
                        <button type="button" class="btn-close" @click="closePostModal">&times;</button>
                    </div>
                    <div class="modal-body">
                        <form @submit.prevent="savePost">
                            <div class="form-group">
                                <label for="postTitle" class="form-label">Title</label>
                                <input type="text" v-model="postForm.title" class="form-control" id="postTitle" required>
                            </div>
                            <div class="form-group">
                                <label for="postContent" class="form-label">Content</label>
                                <textarea v-model="postForm.content" class="form-control" id="postContent" required></textarea>
                            </div>
                            <button type="submit" class="btn btn-primary">{{ editingPost ? 'Update' : 'Create' }}</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
</template>

<script>
import axios from 'axios';

export default {
    data() {
        return {
            posts: [],
            newComment: '',
            showPostModal: false,
            editingPost: false,
            postForm: {
                id: null,
                title: '',
                content: ''
            }
        };
    },
    methods: {
        async fetchPosts() {
            try {
                const response = await axios.get(this.$root.$data.apiUrl + '/posts');
                this.posts = response.data;
            } catch (error) {
                console.error('Error fetching posts:', error);
            }
        },
        async addComment(postId) {
            try {
                await axios.post(this.$root.$data.apiUrl + '/comments', {
                    post_id: postId,
                    content: this.newComment
                });
                this.newComment = '';
                this.fetchPosts(); // Refresh posts to show the new comment
            } catch (error) {
                console.error('Error adding comment:', error);
            }
        },
        async deleteComment(commentId) {
            try {
                await axios.delete(this.$root.$data.apiUrl + '/comments/' + commentId);
                this.fetchPosts(); // Refresh posts to remove the deleted comment
            } catch (error) {
                console.error('Error deleting comment:', error);
            }
        },
        createPost() {
            this.editingPost = false;
            this.postForm = {
                id: null,
                title: '',
                content: ''
            };
            this.showPostModal = true;
        },
        editPost(post) {
            this.editingPost = true;
            this.postForm = {
                id: post.id,
                title: post.title,
                content: post.content
            };
            this.showPostModal = true;
        },
        async savePost() {
            try {
                if (this.editingPost) {
                    await axios.put(this.$root.$data.apiUrl + '/posts/' + this.postForm.id, this.postForm);
                } else {
                    await axios.post(this.$root.$data.apiUrl + '/posts', this.postForm);
                }
                this.closePostModal();
                this.fetchPosts(); // Refresh posts to show the new/updated post
            } catch (error) {
                console.error('Error saving post:', error);
            }
        },
        async deletePost(postId) {
            try {
                await axios.delete(this.$root.$data.apiUrl + '/posts/' + postId);
                this.fetchPosts(); // Refresh posts to remove the deleted post
            } catch (error) {
                console.error('Error deleting post:', error);
            }
        },
        closePostModal() {
            this.showPostModal = false;
        }
    },
    created() {
        this.fetchPosts();
    }
};
</script>

<style scoped>
body {
    background-image: url('https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS5eZUMUR9zIR1egtOHwi43HCsU_eqFVsHiB1sS2G-g_A&s');
    background-size: cover;
    /* Ensure the image covers the entire background */
    background-repeat: no-repeat;
    /* Prevent the image from repeating */
    background-position: center center;
    /* Center the image horizontally and vertically */
}

.navbar {
    background-color: #89b3de;
    padding: 1rem;
}

.modal {
    background: rgba(0, 0, 0, 0.5);
}

.modal-dialog {
    max-width: 500px;
}

.card {
    margin-bottom: 1rem;
}

.input-group {
    margin-top: 1rem;
}

.btn-close {
    background: none;
    border: none;
}

.nav-link {
    color: #ffffff;
    transition: color 0.3s;
    border: 1px solid #0c2846;
    padding: 5px 10px;
    border-radius: 8px;
}

.nav-link:hover {
    color: #042140;
    border-color: #042140;
}

.btn-primary {
    background-color: #4989ce;
    border-color: #021830;
}

.btn-primary:hover {
    background-color: #0056b3;
    border-color: #0056b3;
}

.btn-success {
    background-color: #107a29;
    border-color: #49de6c;
}

.btn-success:hover {
    background-color: #218838;
    border-color: #218838;
}
</style>
