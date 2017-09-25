# Intro to IIIF
## Slide Deck for edUi 2017
[See the slides](http://sdellis.com/intro-to-iiif/). Adapted from Jack Reed's Learn IIIF work.

# Hands On Instructions
For the hands-on section we will install an image server, create a manifest,
and build a custom app with our manifest.

# Exercise 1: Install IIIF Image Server
[Instructions](https://iiif.github.io/training/intro-to-iiif/IIIF_SERVER.html)

# Exercise 2: Create a IIIF Manifest
[Instructions](https://iiif.github.io/training/intro-to-iiif/IIIF_MANIFESTS.html)

# Exercise 3: Create a Slideshow from your Manifest
We are now going to create a slideshow from our manifest.
Prerequisite: [install the latest node and npm via nvm](https://github.com/creationix/nvm).

## Install vue-cli and create a project
```
$ npm install -g vue-cli
$ vue init webpack edui-slideshow
# say 'No' to the the linting and test components to keep it slim for the workshop
$ cd edui-slideshow
$ npm install (have backup copy on flash drives)
$ npm start
```

Open `src/App.vue` and Find/Replace "Hello" with "Slideshow":
```
import Slideshow from './components/Slideshow' // <-- HERE

export default {
  name: 'app',
  components: {
    Slideshow // <-- AND HERE
  }
}
```

Make a copy of `src/components/Hello.vue` and rename it to `Slideshow.vue`.
Open your new file in an editor and paste this into the template at the top:
```
<div>
  <p>
    <a @click="prev">Previous</a> || <a @click="next">Next</a>
  </p>
  <div v-for="number in [currentNumber]">
    <img :src="images[Math.abs(currentNumber) % images.length].display_src"
         v-on:mouseover="stopRotation"
         v-on:mouseout="startRotation"/>

  </div>
</div>
```

Change the name and data properties in the `<script>` section of the component:
```
name: 'slideshow',
data () {
  return {
    images: [],
    currentNumber: 0,
    timer: null
  }
},
```
Now let's add some methods after the data property:
```
methods: {
    startRotation: function() {
      this.timer = setInterval(this.next, 3000);
    },

    stopRotation: function() {
      clearTimeout(this.timer);
      this.timer = null;
    },

    next: function() {
      this.currentNumber += 1
    },
    prev: function() {
      this.currentNumber -= 1
    }
}
```

Finally, let's add our manifest to the project. Copy your manifest.json
that you downloaded into `data/manifest.js`.
```
$ mkdir data
$ cp ~/Desktop/manifest.json data/manifest.js
```

We're going to assign it a variable name and export it as a component. Typically, you would fetch it over the network,
but let's keep it simple for now:

```
var manifest = { your manifest json ... }
export default manifest
```
Now, let's import that at the top of the `<script>` section in our Slideshow component:
```
import manifest from '../data/manifest'
```

Now we need to get the images from our Manifest into our slideshow. Manifesto already has a method
for retrieving the images in a Manifest, as well as many other convenience methods,
so let's leverage that:
```
$ npm install manifesto.js --save
```

Open `src/components/Slideshow` and paste these lines in after the `<script>` tag:
```
import manifest from '../data/manifest'
import manifesto from '../../node_modules/manifesto.js/dist/server/manifesto.js'
const manifestation = manifesto.create(JSON.stringify(manifest))
window.manifestation = manifestation // <-- This allows us to inspect the manifestation in the console
```
Now we can actually play with the "manifestation" in the console:
```
$ npm start
```
In your console (CMD+Opt i), type:
```
manifestation.getLabel()
```
You should see an object containing the label you gave your manifest.

Now type:
```
s = manifestation.getSequences()
s[0].getCanvases()
```
You can see all the canvases with their image urls. It's going to be much easier for us
to transform this data with our own methods that we can "mixin", extending Manifesto with our
own custom methods.

Let's create a file called `mixins/manifesto-vue-mixins.js` and add the following code:

```
const ManifestoVueMixins = {

  mainSequence: function () {
    // mainSequence is the one without an id (not ideal since it could have an id)
    const s = this.getSequences()
    const main_sequence = s.filter((seq) => seq.id !== 'undefined')
    return main_sequence[0]
  },

  getCanvasMainThumb: function (canvas) {
    const images = canvas.getImages()
    const services = images[0].getResource().getServices()
    return services[0].id + '/full/400,/0/default.jpg'
  },

  getCanvasCode: function (canvas_id) {
    const arr = canvas_id.split('/')
    return arr.slice(-1)[0]
  },

  photos: function () {
    const s = this.mainSequence()
    const canvases = s.getCanvases()
    return canvases.map(canvas => ({
      code: this.getCanvasCode(canvas.id),
      caption: canvas.getLabel(),
      id: canvas.id,
      display_src: this.getCanvasMainThumb(canvas)
    }))
  },

}

export default ManifestoVueMixins
```
Now lets import our mixins in the Slideshow.vue component, and extend Manifesto using
`Object.assign`:
```
import mixins from '../mixins/manifesto-vue-mixins'
const manifestation = Object.assign(manifesto.create(JSON.stringify(manifest)), mixins)
```
And add this to the data object:
```
images: manifestation.photos(),
```
Now spin up the app, and we should be seeing a slideshow!
```
npm start
```

# Exercise 4: Create a IIIF Book App with manifestation-vue (a Vue.js component library)
We will now use some prebuilt Vue.js components and mixins that makes it
easier to build IIIF apps quickly.

## Install vue-cli and create a project
```
$ cd ..
$ vue init webpack edui-book
# say 'No' to the the linting and test components to keep it slim for the workshop
$ cd edui-book
$ npm install (have backup copy on flash drives)
$ npm start
```

## Add our IIIF helper libraries

### Manifesto
Manifesto is a library of convenience methods for working with IIIF Resources.
```
$ npm install manifesto.js --save
```
### Manifestions
Manifestations are prebuilt component libraries that make it easy to work
with IIIF Resources (via Manifesto mixins) in the framework of your choice.
```
$ npm install manifestation-vue --save
```

## Add some Data (IIIF manifest)
```
mkdir src/data
curl -o src/data/manifest.js https://rawgit.com/sdellis/edb41a691ea9933bf25ab482b7f6ba2e/raw/a4c5f9155c3e09938bc20345623d3f52fbb21261/manifest.js
```

## Build your app
Open `src/App.vue` and change `import Hello from './components/Hello'` to
```
import {mixins, Tree, Thumbnails} from 'manifestation-vue'
import manifest from './data/manifest'
import manifesto from '../node_modules/manifesto.js/dist/server/manifesto.js'
const manifestation = Object.assign(manifesto.create(JSON.stringify(manifest)), mixins)
```
Change
```
components: {
  Hello
}
```
To
```
components: {
  Tree,
  Thumbnails
},
data() {
  return {
    manifestation: manifestation
  }
}
```
Copy/Paste the following:
```
<header>
  <span>IIIF Vue Components</span>
</header>
<main>
  <tree class="sidebar" v-bind:toc="manifestation.getVueTree()"></tree>
  <thumbnails class="content" v-bind:thumbnails="manifestation.photos()"></thumbnails>
</main>
```
Over everything between the "app" div:
```
<div id="app"></div>
```
Replace the styles with our new layout:
```
<style>
body {
  margin: 0;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

main {
  text-align: left;
  margin: 40px;
}

header {
  margin: 0;
  height: 56px;
  padding: 0 16px 0 24px;
  background-color: #35495E;
  color: #ffffff;
}

header span {
  display: block;
  position: relative;
  font-size: 20px;
  line-height: 1;
  letter-spacing: .02em;
  font-weight: 400;
  box-sizing: border-box;
  padding-top: 16px;
}

.sidebar {
  width: 300px;
  float: left;
}
.content {
  margin-left: 300px;
}
.clear {
  clear: both;
}

</style>
```
## Check it out!
Save what you just did and type the following in the terminal:
```
npm start
```
# Additional Resources
[Learn IIIF](https://www.learniiif.org/) by Jack Reed
[IIIF Workshop](https://iiif.github.io/training/intro-to-iiif/) by Drew Winget and Jack Reed
[Intro to IIIF](http://ronallo.com/iiif-workshop/) by Jason Ronallo
[IIIF Awesome](https://github.com/IIIF/awesome-iiif/blob/master/readme.md)

## License
![](https://www.learniiif.org/assets/by.svg)
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).
