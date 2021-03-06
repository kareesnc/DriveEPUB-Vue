<template>
  <h3 style="margin-left: 2rem">Loading...</h3>
</template>

<script>
// tell eslint that gapi and jquery are defined elsewhere
/*global gapi*/
/*global $*/

export default {
  name: 'DriveFile',
  props: {
    docID: String,
    appData: Object
  },
  emits: [
    'fetched',
    'savedAppData'
  ],
  data () {
    return {
      appData_docID: '',
      bookData: null,
    }
  },
  created: function() {
    this.openBookFile();
  },
  methods: {
    updateAppData: function() {
      // ensure app data doc ID and data exist
      const self = this;
      if(self.appData_docID && self.appData) {
        $.ajax({
          method: 'PATCH',
          url: 'https://www.googleapis.com/upload/drive/v3/files/' + self.appData_docID + '?uploadType=media',
          data: JSON.stringify(self.appData),
          beforeSend: function(xhr) {
            xhr.setRequestHeader('Authorization', 'Bearer ' + gapi.auth.getToken().access_token);
          }
        })
        .done(function() {
          self.$emit('savedAppData');
        })
        .fail(function(jqXHR, textStatus) {
          console.error('Failed to upload book app data. Status: '+textStatus);
          console.log(jqXHR);
        });
      }
      else {
        console.warn('Unable to update book app data');
      }
    },
    createAppData: function() {
      const self = this;
      return new Promise((resolve, reject) => {
        gapi.client.drive.files.create({
          name: self.docID+'.epubcfi',
          mimeType: 'application/json',
          parents: ['appDataFolder']
        }).then(function (response) {
          if(response.result && response.result.id) {
            resolve(response.result.id);
          }
          else {
            console.error('Response not found or did not contain ID');
            console.log(response);
            reject();
          }
        }, function (error) {
          console.error('An error occurred while creating the book app data');
          console.log(error);
          reject();
        });
      });
    },
    fetchAppData: function() {
      const self = this;
      return new Promise((resolve, reject) => {
        gapi.client.drive.files.get({
          fileId: self.appData_docID,
          alt: 'media'
        }).then(function (response) {
          if (response.body && response.headers['Content-Type'] == 'application/json') {
            try {
              resolve(JSON.parse(response.body));
            } catch (e) {
              console.warn('Failed to parse app data');
              console.log(response);
              resolve(null); // continue without app data
            }
          }
          else {
            console.warn('App data file not found, was empty, or was unknown format');
            console.log(response);
            resolve(null); // continue without app data
          }
        }, function (error) {
          console.error('An error occurred while fetching the book app data');
          console.log(error);
          reject();
        });
      });
    },
    getOrCreateAppData: function() {
      const self = this;
      return new Promise((resolve, reject) => {
        gapi.client.drive.files.list({
          q: "name: '"+self.docID+".epubcfi'",
          spaces: 'appDataFolder',
          fields: 'nextPageToken, files(id, name)',
          pageSize: 1
        }).then(function (response) {
          if(response.result && response.result.files.length>0) {
            self.appData_docID = response.result.files[0].id;
            if(response.result.nextPageToken) {
              console.warn('WARNING: There are multiple app data files for this book');
            }
            self.fetchAppData().then((value) => {
              // return book's app data
              resolve(value);
            });
          }
          else {
            console.log('No app data file found, creating...');
            self.createAppData().then((value) => {
              // return nothing; book will use defult values
              // but still don't want to resolve until the new file is created, for update calls
              self.appData_docID = value;
              resolve(null);
            });
          }
        }, function (error) {
          console.error('An error occurred while locating the book app data');
          console.log(error);
          reject();
        });
      });
    },
    fetchDriveFile: function() {
      const self = this;
      return new Promise((resolve, reject) => {
        gapi.client.drive.files.get({
          fileId: self.docID,
          alt: 'media'
        }).then(function (response) {
          if (response.body && response.headers['Content-Type'] == 'application/epub+zip') {
            resolve(response.body);
          }
          else {
            console.error('Response body not found or not epub');
            console.log(response);
            reject();
          }
        }, function (error) {
          console.error('Failed to fetch book from Drive');
          console.log(error);
          reject();
        });
      });
    },
    openBookFile: function() {
      var appDataPromise = this.getOrCreateAppData();
      var bookDataPromise = this.fetchDriveFile();
      Promise.all([appDataPromise, bookDataPromise]).then((values) => {
        const appData = values[0];
        const bookData = values[1];
        this.$emit('fetched',bookData,appData);
      });
    }
  }
}
</script>
