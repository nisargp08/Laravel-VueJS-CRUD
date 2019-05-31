<<template>
        <div>
          <h2>Articles</h2><br>

          <div class="card mb-3">
            <h5 class="card-header">Add Article</h5>
            <div class="card-body">  
          <form @submit.prevent="addArticle()">
            <div class="form-group">
              <label for="title">Title</label>
              <input type="text" v-model="article.title" class="form-control" id="title" placeholder="Enter Article Title">
            </div>
            <div class="form-group">
              <label for="body">Body</label>
              <textarea v-model="article.body" class="form-control" id="body" placeholder="Article Body"></textarea>
            </div>
            <button type="submit" class="btn btn-success">Add Article</button>
          </form>
          </div>
          </div>

          <nav aria-label="Page navigation example">
            <ul class="pagination">
              <li :class="{disabled : pagination.prev_page_url === null}" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Previous</a></li>
              <li class="page-item disabled"><a class="page-link text-dark" href="#">Page <b>{{ pagination.current_page }}</b> of {{ pagination.last_page }}</a></li>
              <li :class="{disabled : pagination.next_page_url === null}" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a></li>
            </ul>
          </nav>
            <div class="card mb-2" v-for="article in articles" :key="article.id">
              <h5 class="card-header">{{ article.title }}</h5>
              <div class="card-body">
                <p class="card-text">{{ article.body }}</p>
                <a href="#" class="btn btn-warning" @click="editArticle(article)">Edit</a>
                <a href="#" class="btn btn-danger" @click="deleteArticle(article.id)">Delete</a>
              </div>
            </div>
        </div>
</template>

<script>
export default {
    data () {
        return{
            articles : [],
            article : {
                id : '',
                title : '',
                body : ''
            },
            article_id : '',
            pagination: {},
            edit : false
        };
    },
    created(){
        this.fetchArticles();
    },
    methods : {
        fetchArticles(page_url) {
          let vm = this;
          page_url = page_url || 'api/articles'
          fetch(page_url)
          .then(res => res.json())
          .then(res => {
            this.articles = res.data;
              vm.makePagination(res.meta, res.links);
          })
          .catch(err => console.log(err));
        },
        makePagination(meta,links) {
          let pagination = {
            current_page : meta.current_page,
            last_page : meta.last_page,
            path : meta.path,
            next_page_url : links.next,
            prev_page_url : links.prev
          }
          this.pagination = pagination;
        },
        deleteArticle(id){
          if(confirm('Are you Sure?')){
            fetch(`api/article/${id}`,{
              method: `delete`
            })
            .then(res => res.json())
            .then(res => {
              alert('Article has been successfully removed');
              this.fetchArticles(this.pagination.path+'?page='+this.pagination.current_page);
            })
            .catch(err => console.log(err));
          }
        },
        addArticle(){
          if(this.edit === false){
            //Add Article
            fetch('api/article',{
              method : 'post',
              body : JSON.stringify(this.article),
              headers : {
                'content-type' : 'application/json'
              }
            })
            .then(res => res.json())
            .then(res => {
              this.article.id = '',
              this.article.body = '',
              alert('Article Successfully added');
              this.fetchArticles();
            })
            .catch(err => console.log(err))
          }
          else{
            //Edit Article
            fetch('api/article',{
              method : 'put',
              body : JSON.stringify(this.article),
              headers : {
                'content-type' : 'application/json'
              }
            })
            .then(res => res.json())
            .then(data => {
              this.article.id = '',
              this.article.body = '',
              this.edit = false;
              alert('Article Successfully updated');
              this.fetchArticles(this.pagination.path+'?page='+this.pagination.current_page);
            })
            .catch(err => console.log(err))
          }
        },
        editArticle(article){
          this.edit = true;
          this.article.id = article.id;
          this.article.article_id = article.id;
          this.article.title = article.title;
          this.article.body = article.body;
        }
    },
}
</script>

