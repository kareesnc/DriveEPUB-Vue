<template>
  <!--The main book area will start hidden, and show once the load is complete-->
  <div id="book" style="visibility: hidden;">
    <div id="toc">
      <button class="btn" title="Open table of contents" disabled>
        <svg class="bi bi-list" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
          <path fill-rule="evenodd" d="M2.5 11.5A.5.5 0 013 11h10a.5.5 0 010 1H3a.5.5 0 01-.5-.5zm0-4A.5.5 0 013 7h10a.5.5 0 010 1H3a.5.5 0 01-.5-.5zm0-4A.5.5 0 013 3h10a.5.5 0 010 1H3a.5.5 0 01-.5-.5z" clip-rule="evenodd"/>
        </svg>
      </button>
      <button class="btn" title="Save current location" disabled>
        <svg width="1em" height="1em" viewBox="0 0 16 16" class="bi bi-bookmark" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
          <path fill-rule="evenodd" d="M8 12l5 3V3a2 2 0 0 0-2-2H5a2 2 0 0 0-2 2v12l5-3zm-4 1.234l4-2.4 4 2.4V3a1 1 0 0 0-1-1H5a1 1 0 0 0-1 1v10.234z"/>
        </svg>
      </button>
      <button class="btn" title="Set font options" disabled>
        <svg width="1em" height="1em" viewBox="0 0 16 16" class="bi bi-type" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
          <path d="m2.244 13.081.943-2.803H6.66l.944 2.803H8.86L5.54 3.75H4.322L1 13.081h1.244zm2.7-7.923L6.34 9.314H3.51l1.4-4.156h.034zm9.146 7.027h.035v.896h1.128V8.125c0-1.51-1.114-2.345-2.646-2.345-1.736 0-2.59.916-2.666 2.174h1.108c.068-.718.595-1.19 1.517-1.19.971 0 1.518.52 1.518 1.464v.731H12.19c-1.647.007-2.522.8-2.522 2.058 0 1.319.957 2.18 2.345 2.18 1.06 0 1.716-.43 2.078-1.011zm-1.763.035c-.752 0-1.456-.397-1.456-1.244 0-.65.424-1.115 1.408-1.115h1.805v.834c0 .896-.752 1.525-1.757 1.525z"/>
        </svg>
      </button>
      <div id="tocList"></div>
    </div>
    <div id="bookContent" class="expanded">
      <div id="viewer"></div>
      <div id="navBtns">
        <button class="btn btn-light" disabled>
          <svg class="bi bi-caret-left-fill" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
            <path d="M3.86 8.753l5.482 4.796c.646.566 1.658.106 1.658-.753V3.204a1 1 0 00-1.659-.753l-5.48 4.796a1 1 0 000 1.506z" />
          </svg>
        </button>
        <button class="btn btn-light" disabled>
          <svg class="bi bi-caret-right-fill" width="1em" height="1em" viewBox="0 0 16 16" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
            <path d="M12.14 8.753l-5.482 4.796c-.646.566-1.658.106-1.658-.753V3.204a1 1 0 011.659-.753l5.48 4.796a1 1 0 010 1.506z" />
          </svg>
        </button>
      </div>
    </div>
  </div>
  <div class="modal-backdrop">
    <div class="modal">
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
          <label>Font size</label>
          <input type="number" value="1.5" />
        </div>
        <div class="form-group">
          <label>Line height</label>
          <input type="number" value="1.5" />
        </div>
        <div class="form-group">
          <label>Size type</label>
          <label><input type="radio" name="sizeType" value="em" checked /> em</label>
          <label><input type="radio" name="sizeType" value="px" /> px</label>
        </div>
        <button class="btn">Update font</button>
        <button class="btn">Clear font</button>
      </div>
    </div>
  </div>
</template>

<script>
// tell eslint that gapi is defined elsewhere
/*global gapi*/
import $ from 'jquery'
// todo: if I don't really need jquery here, consider removing it, and going back to xhr for the app data upload?
//       or just replace it with a smaller, ajax-only library

export default {
  name: 'DriveFile',
  props: {
    book_data: Object,
    app_data: Object
  },
  emits: [
    // todo: app data on location change
    //       font size - here? in another component?
  ],
  data () {
    return {
      book_title: '',
      toc_open: false,
      font_open: false
    }
  },
  created: function() {
  },
  methods: {
  }
}
</script>
