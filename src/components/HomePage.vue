<template>
    <div>
        <!-- Navbar -->
        <nav class="navbar navbar-expand-lg navbar-light bg-light">
            <div class="container-fluid">
                <a class="navbar-brand" href="#">MyBlog</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav"
                    aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="collapse navbar-collapse d-flex justify-content-end" id="navbarNav">
                    <ul class="navbar-nav">
                        <li class="nav-item">
                            <router-link to="/logout" class="nav-link">Logout</router-link>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>

        <!-- Posts Section -->
        <div class="container mt-5">
            <h2 class="mb-4">Posts</h2>
            <div v-for="post in posts" :key="post.id" class="card mb-3 shadow-sm">
                <div class="card-body">
                    <h5 class="card-title">{{ post.title }}</h5>
                    <p class="card-text">{{ post.content }}</p>
                    <p class="card-text"><small class="text-muted">by {{ post.user.name }}</small></p>
                    <div class="d-flex justify-content-between mb-3">
                        <button class="btn btn-primary" @click="editPost(post)">Edit</button>
                        <button class="btn btn-danger" @click="deletePost(post.id)">Delete</button>
                    </div>

                    <!-- Comments Section -->
                    <div class="mt-3">
                        <h6>Comments</h6>
                        <div v-for="comment in post.comments" :key="comment.id" class="mb-2">
                            <p>{{ comment.content }} - <small>{{ comment.user.name }}</small></p>
                            <div class="d-flex justify-content-start">
                                <button class="btn btn-sm btn-primary me-2" @click="editComment(comment)">Edit</button>
                                <button class="btn btn-sm btn-danger" @click="deleteComment(comment.id)">Delete</button>
                            </div>
                        </div>
                        <form @submit.prevent="addComment(post.id)" class="mt-3">
                            <div class="input-group">
                                <input type="text" v-model="newComment" placeholder="Add a comment" class="form-control"
                                    required>
                                <button type="submit" class="btn btn-success">Post</button>
                            </div>
                        </form>
                    </div>
                </div>
            </div>
            <button class="btn btn-success" @click="createPost">Add New Post</button>
        </div>

        <!-- Modal for creating/updating post -->
        <div v-if="showPostModal" class="modal show d-block" tabindex="-1">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">{{ editingPost ? 'Edit Post' : 'New Post' }}</h5>
                        <button type="button" class="btn-close" @click="closePostModal"></button>
                    </div>
                    <div class="modal-body">
                        <form @submit.prevent="savePost">
                            <div class="mb-3">
                                <label for="postTitle" class="form-label">Title</label>
                                <input type="text" v-model="postForm.title" class="form-control" id="postTitle"
                                    required>
                            </div>
                            <div class="mb-3">
                                <label for="postContent" class="form-label">Content</label>
                                <textarea v-model="postForm.content" class="form-control" id="postContent"
                                    required></textarea>
                            </div>
                            <button type="submit" class="btn btn-primary">{{ editingPost ? 'Update' : 'Create'
                                }}</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>

        <!-- Loader -->
        <div v-if="loading" class="loading">
            <div class="spinner-border" role="status">
                <span class="visually-hidden">Loading...</span>
            </div>
        </div>
    </div>
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
            },
            loading: false
        };
    },
    methods: {
        async fetchPosts() {
            this.loading = true;
            try {
                const response = await axios.get(this.$root.$data.apiUrl + '/posts');
                this.posts = response.data;
            } catch (error) {
                console.error('Error fetching posts:', error);
            } finally {
                this.loading = false;
            }
        },

        //Adding comment
        async addComment(postId) {
            try {
                await axios.post(this.$root.$data.apiUrl + '/comments', {
                    post_id: postId,
                    content: this.newComment
                });
                this.newComment = '';
                this.fetchPosts();
            } catch (error) {
                console.error('Error adding comment:', error);
            }
        },

        //Deleting comment
        async deleteComment(commentId) {
            try {
                await axios.delete(this.$root.$data.apiUrl + '/comments/' + commentId);
                this.fetchPosts();
            } catch (error) {
                console.error('Error deleting comment:', error);
            }
        },

        //Creating post
        createPost() {
            this.editingPost = false;
            this.postForm = {
                id: null,
                title: '',
                content: ''
            };
            this.showPostModal = true;
        },

        //Editing post
        editPost(post) {
            this.editingPost = true;
            this.postForm = {
                id: post.id,
                title: post.title,
                content: post.content
            };
            this.showPostModal = true;
        },

        //Saving post
        async savePost() {
            try {
                if (this.editingPost) {
                    await axios.put(this.$root.$data.apiUrl + '/posts/' + this.postForm.id, this.postForm);
                } else {
                    await axios.post(this.$root.$data.apiUrl + '/posts', this.postForm);
                }
                this.closePostModal();
                this.fetchPosts();
            } catch (error) {
                console.error('Error saving post:', error);
            }
        },

        //Deleting post
        async deletePost(postId) {
            try {
                await axios.delete(this.$root.$data.apiUrl + '/posts/' + postId);
                this.fetchPosts();
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
.navbar {
    background-color: #f8f9fa;
    padding: 1rem;
}

.card {
    border-radius: 10px;
}

.card-body {
    padding: 1.5rem;
}

.btn {
    border-radius: 5px;
    transition: background-color 0.3s ease;
}

.btn-primary {
    background-color: #007bff;
    color: #fff;
}

.btn-primary:hover {
    background-color: #0056b3;
}

.btn-danger {
    background-color: #dc3545;
    color: #fff;
}

.btn-danger:hover {
    background-color: #bd2130;
}

.btn-success {
    background-color: #28a745;
    color: #fff;
}

.btn-success:hover {
    background-color: #218838;
}

.modal {
    background: rgba(0, 0, 0, 0.5);
}

.modal-dialog {
    max-width: 500px;
}

.loading {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 1050;
    background: rgba(255, 255, 255, 0.8);
    padding: 2rem;
    border-radius: 0.5rem;
}

.spinner-border {
    width: 3rem;
    height: 3rem;
}
</style>
