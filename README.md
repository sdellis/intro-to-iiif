# Intro to IIIF
## Slide Deck for edUi 2017
Adapted from Jack Reed's Learn IIIF work.

# Hands On Instructions
For the hands-on section we will install an image server, create a manifest,
and build a custom app with our manifest.

# Install IIIF Image Server
Instructions go here.

# Create a IIIF Manifest
Instructions go here.

# Instructions for IIIF Vue App

## Install vue-cli and create a project
```
$ npm install -g vue-cli
$ vue init webpack my-project
# say 'No' to the the linting and test components to keep it slim for the workshop
$ cd my-project
$ npm install (have backup copy on flash drives)
$ npm run dev
```

## Add our IIIF helper scripts

### Manifesto
Manifesto is a library of convenience methods for working with IIIF Resources
```
$ npm install manifesto.js --save
```
### Manifestions
Manifestations are prebuilt libraries of components that make it easy to work
with IIIF Resources (via Manifesto mixins) in the framework of your choice
```
$ npm install manifestation-vue --save
```

## Add some Data (IIIF manifest)
```
mkdir src/data
cd src/data
curl -o src/data/manifest.js https://rawgit.com/sdellis/edb41a691ea9933bf25ab482b7f6ba2e/raw/a4c5f9155c3e09938bc20345623d3f52fbb21261/manifest.js
```

## Build your app
Open src/App.vue and change `import Hello from './components/Hello'` to
```
import {mixins, Tree, Title, Thumbnails} from 'manifestation-vue'
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

## Check it out!
Save what you just did and type the following in the terminal:
```
npm start
```


## License
Creative Commons Attribution
