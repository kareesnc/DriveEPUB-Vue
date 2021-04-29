# Drive EPUB (Vue.js)

Read EPUB files in Google Drive without downloading them!  
  
This app requests sensitive Drive OAuth scopes and cannot be fully run locally - 
you must install it on a server or hosting platform with a host name and SSL certificate, 
and configure the OAuth consent screen for your project. You must also set the client ID 
and api key in the .env file to enable access to your API.  
  
You can test the EPUB rendering portions locally, using the demo file setting in the .env file.  

Users must manually visit the application home page and authorized it for the first time. 
Afterwards, the application should show in the context menu of epub files.  

Required OAuth scopes:  
* ../auth/drive.file  
* ../auth/drive.appdata  

Drive integration:  
* Open URL: https://your.host/index.html  
* MIME type: application/epub+zip  


## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
