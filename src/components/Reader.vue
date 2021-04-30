<template>
  <h3 v-if="!rendered" style="margin-left: 2rem; position: absolute;">Loading...</h3>
  <div id="book" :class="{'rendered': rendered}">
    <div id="toc" :class="{'expanded': tocOpen}">
      <button class="btn" @click.prevent="toggleToc" title="Open table of contents">
        <svg class="bi bi-list" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
          <path fill-rule="evenodd" d="M2.5 11.5A.5.5 0 013 11h10a.5.5 0 010 1H3a.5.5 0 01-.5-.5zm0-4A.5.5 0 013 7h10a.5.5 0 010 1H3a.5.5 0 01-.5-.5zm0-4A.5.5 0 013 3h10a.5.5 0 010 1H3a.5.5 0 01-.5-.5z" clip-rule="evenodd"/>
        </svg>
      </button>
      <button class="btn" @click.prevent="saveAppDataFile" title="Save current location" :disabled="disableSave">
        <svg width="1em" height="1em" viewBox="0 0 16 16" class="bi bi-bookmark" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
          <path fill-rule="evenodd" d="M8 12l5 3V3a2 2 0 0 0-2-2H5a2 2 0 0 0-2 2v12l5-3zm-4 1.234l4-2.4 4 2.4V3a1 1 0 0 0-1-1H5a1 1 0 0 0-1 1v10.234z"/>
        </svg>
      </button>
      <button class="btn" @click.prevent="openFontModal" title="Set font options">
        <svg width="1em" height="1em" viewBox="0 0 16 16" class="bi bi-type" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
          <path d="m2.244 13.081.943-2.803H6.66l.944 2.803H8.86L5.54 3.75H4.322L1 13.081h1.244zm2.7-7.923L6.34 9.314H3.51l1.4-4.156h.034zm9.146 7.027h.035v.896h1.128V8.125c0-1.51-1.114-2.345-2.646-2.345-1.736 0-2.59.916-2.666 2.174h1.108c.068-.718.595-1.19 1.517-1.19.971 0 1.518.52 1.518 1.464v.731H12.19c-1.647.007-2.522.8-2.522 2.058 0 1.319.957 2.18 2.345 2.18 1.06 0 1.716-.43 2.078-1.011zm-1.763.035c-.752 0-1.456-.397-1.456-1.244 0-.65.424-1.115 1.408-1.115h1.805v.834c0 .896-.752 1.525-1.757 1.525z"/>
        </svg>
      </button>
      <div id="tocList" v-if="tocOpen">
        <a 
          href="#" 
          v-for="chapter in tocList" 
          :key="chapter.id"
          @click.prevent="rendition.display(chapter.href)"
        >
          {{chapter.label}}
        </a>
      </div>
    </div>
    <div id="bookContent" :class="{'expanded': !tocOpen}">
      <div id="viewer"></div>
      <div id="navBtns">
        <button class="btn btn-light" @click.prevent="goLeft" :disabled="disableLeft">
          <svg class="bi bi-caret-left-fill" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
            <path d="M3.86 8.753l5.482 4.796c.646.566 1.658.106 1.658-.753V3.204a1 1 0 00-1.659-.753l-5.48 4.796a1 1 0 000 1.506z" />
          </svg>
        </button>
        <button class="btn btn-light" @click.prevent="goRight" :disabled="disableRight">
          <svg class="bi bi-caret-right-fill" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
            <path d="M12.14 8.753l-5.482 4.796c-.646.566-1.658.106-1.658-.753V3.204a1 1 0 011.659-.753l5.48 4.796a1 1 0 010 1.506z" />
          </svg>
        </button>
      </div>
    </div>
  </div>
  <div id="modal-backdrop">
    <div id="fontModal" class="modal">
      <div class="modal-header">
        <button type="button" class="close">
          <span aria-hidden="true">&times;</span>
        </button>
        <h3 class="modal-title">Set font options</h3>
      </div>
      <div class="modal-body">
        <p style="margin-top: 0">This will be saved along with your reading progress for this book.</p>
        <div class="form-group">
          <label>Select a font:</label>
          <select>
            <option value="Verdana">Verdana (sans-serif)</option>
            <option value="Georgia">Georgia (serif)</option>
            <option value="Courier New">Courier New (mono)</option>
          </select>
        </div>
        <div class="form-group">
          <label>Or enter a custom font:</label>
          <input type="text" />
        </div>
        <div class="form-group">
          <label>Font size:</label>
          <input type="number" value="1.5" />
        </div>
        <div class="form-group">
          <label>Line height:</label>
          <input type="number" value="1.5" />
        </div>
        <div class="form-group">
          <label>Size type:</label>
          <label><input type="radio" name="sizeType" value="em" checked /> em</label>
          <label><input type="radio" name="sizeType" value="px" /> px</label>
        </div>
      </div>
      <div class="modal-footer">
        <button class="btn">Update font</button>
        <button class="btn">Clear font</button>
      </div>
    </div>
  </div>
</template>

<script>
import $ from 'jquery'
import ePub from 'epubjs'

export default {
  name: 'Reader',
  props: {
    bookData: ArrayBuffer,
    appData: Object
  },
  emits: [
    'updateAppDataContent',
    'saveAppDataFile'
  ],
  data () {
    return {
      book: null,
      rendition: null,
      tocList: null,
      rendered: false,
      tocOpen: false,
      fontOpen: false,
      disableLeft: true,
      disableRight: true,
      disableSave: false
    }
  },
  mounted: function() {
    this.openBook();
    const self = this;
    $(window).resize(function() {
      if(self.book) {
        self.book.refreshBook();
      }
    });
    // Modal close functions
    $(".modal .close").on("click",this.closeModals);
    $("#modal-backdrop").on("click",function(e){
      if(e.target.id=="modal-backdrop") {
        self.closeModals();
      }
    });
  },
  methods: {
    openBook: function() {
      this.book = ePub();
      this.book.open(this.bookData, "binary");
      // Set height/width of the viewer
      const width = document.getElementById("bookContent").offsetWidth-10; // ??? Chrome
      const height = document.getElementById("bookContent").offsetHeight-60; // save room for nav buttons
      // Render book
      this.rendition = this.book.renderTo("viewer", {
        width: width+"px",
        height: height+"px",
        spread: "always"
      });
      if(this.appData && this.appData.cfi) {
        this.rendition.display(this.appData.cfi);
      }
      else {
        // If location isn't specified
        this.rendition.display();
      }

      const self = this;
      this.book.ready.then(function() {
        self.bookPostLoad();
        self.applyFont();
        self.rendered = true;
      });

      // Render TOC
      this.book.loaded.navigation.then(function (toc) {
        if(toc && toc.toc) {
          self.tocList = toc.toc;
        }
      });

      // Handle relocates
      this.rendition.on("relocated", function(location) {
          self.relocated(location);
      });

      this.rendition.on("layout", function (layout) {
        let viewer = document.getElementById("viewer");
        if (layout.spread) {
          viewer.classList.remove("single");
        } else {
          viewer.classList.add("single");
        }
      });

      // Handle unload
      window.addEventListener("unload", function() {
        self.destroyBook();
      });
    },
    bookPostLoad: function() {
      // Add book title to window title
      const title = this.book.package.metadata.title;
      if(!document.title.includes(" - "+title)) {
          document.title += " - "+title;
      }

      // Add key listeners
      const keyListener = function (e) {
        if ((e.keyCode || e.which) == 37) { // Left Key
          this.goLeft();
        }
        if ((e.keyCode || e.which) == 39) { // Right Key
          this.goRight();
        }
      };
      this.rendition.on("keyup", keyListener);
      $(document).on("keyup", keyListener);
    },
    goLeft: function() {
      if(this.book.package.metadata.direction === "rtl") {
        this.rendition.next();
      }
      else {
        this.rendition.prev();
      }
    },
    goRight: function() {
      if(this.book.package.metadata.direction === "rtl") {
        this.rendition.prev();
      }
      else {
        this.rendition.next();
      }
    },
    relocated: function(location) {
      // Enable/disable previous/next buttons when at start/end of book
      if(this.book.package.metadata.direction === "rtl") {
        this.disableLeft = location.atEnd;
        this.disableRight = location.atStart;
      }
      else {
        this.disableLeft = location.atStart;
        this.disableRight = location.atEnd;
      }
      // Store new location
      this.updateAppDataContent('cfi',location.start.cfi);
    },
    updateAppDataContent: function(property,value) {
      this.$emit('updateAppDataContent',property,value);
    },
    saveAppDataFile: function() {
      this.disableSave = true;
      this.$emit('saveAppDataFile');
    },
    enableSaveButton: function() {
      this.disableSave = false;
    },
    applyFont: function() {
      if(this.appData && this.appData.fontFamily && this.appData.fontSize && this.appData.lineHeight) {
        this.book.rendition.themes.default({ 
          "p": { 
            "font-family": this.appData.fontFamily,
            "font-size": this.appData.fontSize,
            "line-height": this.appData.lineHeight
          }
        });
      }
    },
    setFont: function(family,size,lineHeight) {
      this.updateAppDataContent('fontFamily',family);
      this.updateAppDataContent('fontSize',size);
      this.updateAppDataContent('lineHeight',lineHeight);
      this.applyFont();
    },
    clearFont: function() {
      this.updateAppDataContent('fontFamily','');
      this.updateAppDataContent('fontSize','');
      this.updateAppDataContent('lineHeight','');
      this.refreshBook();
    },
    refreshBook: function() {
      this.destroyBook();
      this.openBook();
    },
    destroyBook: function() {
      if(this.book) {
        this.book.destroy();
        this.rendered = false;
      }
    },
    toggleToc: function() {
      if(this.tocOpen) {
        this.tocOpen = false;
      }
      else {
        this.tocOpen = true;
      }
      this.refreshBook();
    },
    closeModals: function() {
      $("#modal-backdrop").css("display","none");
      $(".modal").css("display","none");
    },
    openFontModal: function() {
      $("#modal-backdrop").css("display","block");
      $("#fontModal").css("display","block");
    }
  }
}
</script>

<style scoped>
  #book {
    height: 100%;
    display: flex;
    align-items: stretch;
    opacity: 0;
  }
  #book.rendered {
    opacity: 1;
  }
  #toc {
    width: 60px;
  }
  #toc.expanded {
    min-width: 200px;
    flex-basis: 20%;
    height: 100%;
    overflow-y: auto;
  }
  #toc .btn {
    padding: 5px 10px;
    font-size: 16px;
    margin: 10px 10px 5px 10px;
    line-height: 10px;
    border-radius: 5px;
  }
  #toc a {
    display: block;
    margin-left: 10px;
    color: #0070e8;
    text-decoration: none;
  }
  #toc a:hover {
    color: #0056b3;
    text-decoration: underline;
  }
  #bookContent {
    flex-basis: 80%;
  }
  #bookContent.expanded {
    flex-basis: 100%;
  }
  #bookLoading, #error, .authorize {
    position: absolute;
    top: 15px;
    left: 15px;
    z-index: 999;
  }
  #error {
    background: #ffbbbb;
    padding: 10px 15px 0px 15px;
    border-radius: 15px;
  }
  #error p {
    margin: 0 0 10px 0;
  }
  #navBtns {
    display: flex;
    height: 50px;
    margin-right: 10px;
  }
  #navBtns > .btn {
    flex-basis: 50%;
    font-size: 30px;
    padding: 0;
    line-height: 1;
    border-radius: 5px;
  }
  .btn {
    cursor: pointer;
  }
  .btn.disabled, .btn:disabled {
    cursor: unset;
  }
  .form-group {
    margin-bottom: 10px;
  }
  .form-group label {
    margin-right: 10px;
  }
  #modal-backdrop {
    display: none;
    position: fixed;
    z-index: 1;
    padding-top: 100px;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0,0,0,0.4);
  }
  .modal {
    display: none;
    background-color: #fefefe;
    margin: auto;
    border: 1px solid #888;
    border-radius: 10px;
    width: 100%;
    max-width: 600px;
  }
  .modal-header {
    padding: 15px 15px 10px 15px;
    border-bottom: 1px solid #888;
  }
  .modal-header h3 {
    margin: 0;
  }
  .modal-body {
    padding: 15px;
  }
  .modal-footer {
    padding: 15px;
  }
  .modal .btn {
    padding: 5px;
    margin-right: 10px;
  }
  .close {
    color: #aaaaaa;
    float: right;
    font-size: 20px;
    font-weight: bold;
    border-radius: 5px;
  }
  .close:hover, .close:focus {
    color: #000;
    text-decoration: none;
    cursor: pointer;
  }
</style>
