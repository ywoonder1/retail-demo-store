<template>
  <div class="content">

    <!-- Categories Navigation -->
    <div class="container mb-4">
      <div class="container mb-4" v-if="!categories.length">
        <i class="fas fa-spinner fa-spin fa-3x"></i>
      </div>
      <div class="row col-sm-12 col-md-12 col-lg-12 d-none d-sm-block">
        <ul class="nav nav-pills nav-fill mx-auto">
          <li class="nav-item">
            <router-link class="nav-link" :to="{name:'CategoryDetail', params: {id: 'featured'}}" v-bind:class="{ active: display == 'featured' }">Featured</router-link> 
          </li>
          <li class="nav-item" v-for="category in categories" v-bind:key=category.id>
          <router-link class="nav-link" :to="{name:'CategoryDetail', params: {id: category.name}}" v-bind:class="{ active: display == category.name }">{{ category.name | capitalize }}</router-link> 
          </li>
        </ul>
      </div>
      <div class="dropdown show d-block d-sm-none">
        <a class="btn dropdown-toggle btn-primary" href="#" role="button" id="dropdownMenuLink" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
          Select Category
        </a>
        <div class="dropdown-menu" aria-labelledby="dropdownMenuLink">
          <router-link class="dropdown-item" :to="{name:'CategoryDetail', params: {id: 'featured'}}" v-bind:class="{ active: display == 'featured' }">Featured</router-link> 
          <router-link class="dropdown-item" v-for="category in categories" v-bind:key="category" :to="{name:'CategoryDetail', params: {id: category.name}}" v-bind:class="{ active: display == category.name }">
            {{ category.name | capitalize }}
          </router-link> 
        </div>
      </div>
    </div>

    <!-- Announcements -->
    <div id="featuredProducts" class="carousel slide col-sm-12 col-md-12 col-lg-12 d-none d-md-block mb-5" data-ride="carousel">
      <ol class="carousel-indicators">
        <li data-target="#featuredProducts" data-slide-to="0" class="active"></li>
        <li data-target="#featuredProducts" data-slide-to="1"></li>
        <li data-target="#featuredProducts" data-slide-to="2"></li>
      </ol>
      <div class="carousel-inner">
        <div class="carousel-item active">
          <img class="d-block w-100" v-bind:src="imageRootURL + 'apparel/2.jpg'" alt="First slide">
            <div class="carousel-caption">
              <h5>The Apparel Collection</h5>
              <p>Cozy sweaters and classic styles for your wardrobe.</p>
              <router-link class="btn btn-outline-light" :to="{name:'CategoryDetail', params: {id: 'apparel'}}">See More</router-link> 
            </div>
        </div>
        <div class="carousel-item">
          <img class="d-block w-100" v-bind:src="imageRootURL + 'outdoors/4.jpg'" alt="Second slide">
            <div class="carousel-caption">
              <h5>Lets Go Fishing</h5>
              <p>Start gearing up for summer adventures with our new fishing gear!</p>
              <router-link class="btn btn-outline-light" :to="{name:'CategoryDetail', params: {id: 'outdoors'}}">See More</router-link> 
            </div>
        </div>
        <div class="carousel-item">
          <img class="d-block w-100" v-bind:src="imageRootURL + 'beauty/3.jpg'" alt="Third slide">
              <div class="carousel-caption">
              <h5>Beauty Products</h5>
              <p>Popular beauty products now back in stock.</p>
              <router-link class="btn btn-outline-light" :to="{name:'CategoryDetail', params: {id: 'beauty'}}">See More</router-link> 
            </div>
        </div>
      </div>
      <a class="carousel-control-prev" href="#featuredProducts" role="button" data-slide="prev">
        <span class="carousel-control-prev-icon" aria-hidden="true"></span>
        <span class="sr-only">Previous</span>
      </a>
      <a class="carousel-control-next" href="#featuredProducts" role="button" data-slide="next">
        <span class="carousel-control-next-icon" aria-hidden="true"></span>
        <span class="sr-only">Next</span>
      </a>
    </div>

    <!-- Category List -->
    <div class="container">
      <h4>Categories</h4>
      <div class="container mb-4" v-if="!categories.length">
        <i class="fas fa-spinner fa-spin fa-3x"></i>
      </div>
      <div class="row">
        <div class="card-deck col-sm-12 col-md-12 col-lg-12 mt-4">
          <Category v-for="category in categories" 
            v-bind:key="category.id"
            :category="category"
          />
        </div>
      </div>
    </div>

    <!-- Recommended/Featured Product List -->
    <div class="container mt-5 user-recommendations" v-if="userID">
      <h4>{{ this.display | capitalize }}</h4>
      <div class="container mb-4" v-if="!user_recommended.length">
        <i class="fas fa-spinner fa-spin fa-3x"></i>
      </div>
      <div class="row">
        <div class="card-deck col-sm-12 col-md-12 col-lg-12 mt-4">
          <Product v-for="recommendation in user_recommended" 
            v-bind:key="recommendation.product.id"
            :product="recommendation.product"
            :experiment="recommendation.experiment"
          />
        </div>
      </div>
    </div>
    <div class="container guest-recommendations" v-if="!userID">
      <h4>{{ this.display | capitalize }}</h4>
      <div class="container mb-4" v-if="!guest_recommended.length">
        <i class="fas fa-spinner fa-spin fa-3x"></i>
      </div>
      <div class="row">
        <div class="card-deck col-sm-12 col-md-12 col-lg-12 mt-4">
          <Product v-for="product in guest_recommended" 
            v-bind:key="product.id"
            :product="product"
          />
        </div>
      </div>
    </div>
  </div> 
</template>

<script>
import AmplifyStore from '@/store/store'

import { RepositoryFactory } from '@/repositories/RepositoryFactory'
import { AnalyticsHandler } from '@/analytics/AnalyticsHandler'

import Product from './components/Product.vue'
import Category from './components/Category.vue'

const ProductsRepository = RepositoryFactory.get('products')
const RecommendationsRepository = RepositoryFactory.get('recommendations')
const MaxRecommendations = 9
const ExperimentFeature = 'home_product_recs'

export default {
  name: 'Main',
  components: {
    Product,
    Category
  },
  data() {
    return {
      categories: [],
      guest_recommended: [],
      user_recommended: [],
      errors: [],
      display: ''
    }
  },
  async created () {
    this.getRecommendations()
    this.getCategories()
  },
  methods: {
    async getRecommendations() {
      if (this.userID) {
        this.display = 'Inspired by your shopping trends'
        this.getUserRecommendations()
      }
      else {
        this.display = 'featured'
        const { data } = await ProductsRepository.getFeatured()
        this.guest_recommended = data
      }
    },
    async getUserRecommendations() {
      const { data } = await RecommendationsRepository.getRecommendationsForUser(this.userID, '', MaxRecommendations, ExperimentFeature)

      this.user_recommended = data

      if (this.user_recommended.length > 0 && 'experiment' in this.user_recommended[0]) {
        AnalyticsHandler.identifyExperiment(this.user_recommended[0].experiment)
      }
    },
    async getCategories () {
      const { data } = await ProductsRepository.getCategories()
      this.categories = data
    }
  },
  computed: {
    user() { 
      return AmplifyStore.state.user
    },
    userID() { 
      return AmplifyStore.state.userID
    },
    imageRootURL() {
      return process.env.VUE_APP_IMAGE_ROOT_URL ? process.env.VUE_APP_IMAGE_ROOT_URL : '/images/'
    }
  },
  filters: {
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
}
</script>

<style scoped>
  .content {
    padding-top: 1rem;
  }

  .carousel {
    max-height: 600px;
  }

  .carousel-item {
    max-height: 600px;
  }
</style>