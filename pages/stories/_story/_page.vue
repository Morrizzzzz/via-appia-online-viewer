<template>
  <div>
    <div class="flex mb-4">
      <div class="flex-grow" />

      <NuxtLink
        :disabled="!prev"
        class="btn"
        :to="{ to: 'stories', params: {story: $route.params.story, page: prev && prev.slug}}"
      >
        Back
      </NuxtLink>

      <NuxtLink
        :disabled="!next"
        class="btn ml-4"
        :to="{ to: 'stories', params: {story: $route.params.story, page: next && next.slug}}"
      >
        Next
      </NuxtLink>
    </div>

    <!--    page time line -->
    <div class="fixed top-48 right-0">
      <steps-timeline-links :pages="pages" />
    </div>
    <!--      <div class="bg-gray-700 bg-opacity-90 rounded p-4">-->
    <!--                <div class="text-xl font-bold ">-->
    <!--                  {{ page && page.title }}-->
    <!--                </div>-->
    <!--                <div class="w-full mt-3">-->
    <!--                  <nuxt-content :document="page" class="prose-sm sm:prose" />-->
    <!--                </div>-->
    <!--      </div>-->
  </div>
</template>

<script>

import { potreeRef } from '~/api/VAPotree'
import { loadVideo, videos, removeVideo } from '~/api/videos'
// import AppSettings from '~/content/app-settings.yaml'

export default {
  setup () {
    return { videos, potreeRef, removeVideo, THREE }
  },
  data () {
    return {
      error: false,
      pages: null,
      page: null,
      prev: null,
      next: null
    }
  },
  async fetch () {
    const params = this.$route.params

    // Page details
    this.page = await this.$content(params.story, params.page)
      .fetch()
      .catch((err) => { console.error({ statusCode: 404, message: 'Page not found', error: err }) })

    // Get the list of pages of the story
    this.pages = await this.$content(params.story)
      .sortBy('slug', 'asc')
      .only(['title', 'description', 'path'])
      .fetch()
      .catch((err) => { console.error({ statusCode: 404, message: 'Page not found', error: err }) })

    // Next and previous pages links
    const [prev, next] = await this.$content(params.story)
      .sortBy('slug', 'asc')
      .only(['title', 'slug'])
      .surround(params.page)
      .fetch()
      .catch((err) => { console.error({ statusCode: 404, message: 'Page not found', error: err }) })

    this.prev = prev
    this.next = next
  },
  watch: {
    // When changing pages, refetch the content page and reload the method
    $route () {
      // delete previous video on the page
      if (this.page.mediaPath) {
        removeVideo(this.page.mediaPath)
      }
      console.log('🎹 before leave?', this.page.mediaPath)
      this.$fetch().then(() => {
        this.initPagePosition()
      })
    // this.getAnimationPaths(to.params)
    }
  },
  mounted () {
    this.$fetch().then(() => {
      this.initPagePosition()
    })
  },
  methods: {
    initPagePosition () {
      // // Cat image todo delete
      // this.loadImage()

      // goToCameraPosition
      potreeRef.viewer.setFOV(this.page?.cameraFOV || 60)
      potreeRef.viewer.scene.view.setView(
        this.page.cameraPosition,
        this.page.cameraTarget,
        this.page.animationEntry || 2000, () => {
          console.log('🎹 eneded movement')
        })

      // Load media: video or static image
      loadVideo(this.page)
    },

    loadImage () {
      const scene = potreeRef.viewer.scene.scene
      // Remove previous image here
      // TODO
      // console.log('🎹', potreeRef.viewer?.scene.uuid)
      if (this.page?.image) {
        /**
         * Video
         **/
        // Create a texture loader so we can load our image file
        const loader = new THREE.TextureLoader()
        // Load an image file into a custom material
        const material = new THREE.MeshLambertMaterial({
          map: loader.load('https://s3.amazonaws.com/duhaime/blog/tsne-webgl/assets/cat.jpg')
        })
        // create a plane geometry for the image with a width of 10
        // and a height that preserves the image's aspect ratio
        const geometry = new THREE.PlaneGeometry(10, 10 * 0.75)
        // combine our image geometry and material into a mesh
        const mesh = new THREE.Mesh(geometry, material)
        // set the position of the image mesh in the x,y,z dimensions
        mesh.position.set(296255.77195538126, 4633698.280614582, 139.53239533881214)
        mesh.rotation.set(90, 0, 0)
        // add the image to the scene
        scene.add(mesh)
      }
    }
  }
}
</script>
