<template>
  <div>
    	<h1>{{ appName }}</h1>
    	<p>{{ appDescription }}</p>

		<div style="margin-top: 10px;">
			<g-signin-button :params="googleSignInParams" id="btnSignIn" @success="onSignInSuccess" @error="onSignInError">
    			<i class="icon fa-google"></i>&nbsp;&nbsp;&nbsp;Sign in with Google
  			</g-signin-button>
		</div>

		<div class="container" v-if="isUserLoggedIn">
			<!--UPLOAD-->
			<form enctype="multipart/form-data" novalidate v-if="isInitial || isSaving || isSuccess || isFailed">
				<h1>Upload images</h1>
				<div class="dropbox">
					<input type="file" multiple :name="uploadFieldName" :disabled="isSaving" @change="filesChange($event.target.name, $event.target.files)" accept="image/*" class="input-file">
					<p v-if="isInitial || isSuccess || isFailed">
						Drag your file(s) here to begin
					</p>
					<p v-if="isSaving">
						Uploading {{ fileCount }} files...
					</p>
				</div>
			</form>
		</div>
					
		<ul class="icons" style="margin-top: 40px;" v-if="!isUserLoggedIn">
			<li><a href="https://twitter.com/goh_chunlin" class="icon fa-twitter" target="_blank"><span class="label">Twitter</span></a></li>
			<li><a href="https://github.com/goh-chunlin" class="icon fa-github" target="_blank"><span class="label">Github</span></a></li>
			<li><a href="mailto:gclin009@gmail.com" class="icon fa-envelope-o"><span class="label">Email</span></a></li>
		</ul>
  </div>
</template>

<script>
	//import { upload } from 'upload-service.js';

	import * as axios from 'axios';

	const url = 'https://changsha.azurewebsites.net/api/HttpTriggerCSharp1?code=y1JhE4zZy9X7SnAbjXLAySI0tx2/pRcY2NnoMdK/gj8aE75QfikKAA==';
    
	const STATUS_INITIAL = 0, STATUS_SAVING = 1, STATUS_SUCCESS = 2, STATUS_FAILED = 3;

  	export default {
		name: 'home',

		data () {
			return {
				appName: 'Changsha',
				appDescription: 'Please login to proceed',
				googleSignInParams: {
					client_id: '982369481425-e52ekf9sk58dgusi2kojf4fii1b138d8.apps.googleusercontent.com'
				},

				currentLoggedInUserName: '',
				fileCount: 0,
				uploadedFiles: [],
				uploadError: null,
				currentStatus: null,
				uploadFieldName: 'photos'
			}
		},

		created () {
			this.hideGallery();

			this.mounted();
		},

		updated () {
			if (this.currentLoggedInUserName != '') {
				axios.get(url)
				// get data			
				.then(function (response) {
					console.log(response.data);

					var photoNames = response.data.photoNames.split(",");
					var createdBys = response.data.createdBys.split(",");
					var createdAts = response.data.createdAts.split(",");

					var gallery = '';
					for(var i = 0; i < photoNames.length; i++){
						gallery += `<article>
							<a class="thumbnail" href="https://changsha.blob.core.windows.net/changsha/` + photoNames[i] + `" data-position="left center"><img src="https://changsha.blob.core.windows.net/changsha/` + photoNames[i] + `" alt="" /></a>
							<h2>Submitted by ` + createdBys[i] + `</h2>
							<p>Date: ` + createdAts[i]  + `</p>
						</article>`;
					}

					$('#thumbnails').html(gallery);

					main.init();
				});
			}
		},

		computed: {
			isUserLoggedIn() {
				return this.currentLoggedInUserName != '';
			},
			isInitial() {
				return this.currentStatus === STATUS_INITIAL;
			},
			isSaving() {
				return this.currentStatus === STATUS_SAVING;
			},
			isSuccess() {
				return this.currentStatus === STATUS_SUCCESS;
			},
			isFailed() {
				return this.currentStatus === STATUS_FAILED;
			}
		},

		methods: {
			hideGallery () {
				$('#viewer').hide();
				$('#thumbnails').hide();
			},
			onSignInSuccess (googleUser) {
				// `googleUser` is the GoogleUser object that represents the just-signed-in user. 
				// See https://developers.google.com/identity/sign-in/web/reference#users 
				const profile = googleUser.getBasicProfile() // etc etc 
				this.currentLoggedInUserName = profile.U3;
				this.appName = profile.ig;
				this.appDescription = 'Your trip photos gallery',
				$('#btnSignIn').hide();
				$('#viewer').show();
				$('#thumbnails').show();
				$('html').removeClass('html-class');

				console.log(profile);
    		},
    		onSignInError (error) {
      			// `error` contains any error occurred. 
      			console.log('OH NOES', error)
			},
			
			reset() {
				// reset form to initial state
				this.currentStatus = STATUS_INITIAL;
				this.uploadedFiles = [];
				this.uploadError = null;
			},
			save(formData) {
				// upload data to the server
				this.currentStatus = STATUS_SAVING;

				this.upload(formData)
				.then(x => {
					this.uploadedFiles = [].concat(x);
					this.currentStatus = STATUS_SUCCESS;
				})
				.catch(err => {
					this.uploadError = err.response;
					this.currentStatus = STATUS_FAILED;
				});
			},
			filesChange(fieldName, fileList) {
				// handle file changes
				const formData = new FormData();

				if (!fileList.length) return;

				this.fileCount = fileList.length;

				// append the files to FormData
				Array
				.from(Array(fileList.length).keys())
				.map(x => { formData.append(fieldName, fileList[x], fileList[x].name); });

				// save it
				this.save(formData);
			},
			mounted() {
				this.reset();
			},
			upload(formData) {
    			return axios.post(url + '&username=' + this.currentLoggedInUserName, formData)
				// get data			
				.then(function (response) {
					console.log(response.data);

					var photoNames = response.data.photoNames.split(",");
					var createdBys = response.data.createdBys.split(",");
					var createdAts = response.data.createdAts.split(",");

					var gallery = '';
					for(var i = 0; i < photoNames.length; i++){
						gallery += `<article>
							<a class="thumbnail" href="https://changsha.blob.core.windows.net/changsha/` + photoNames[i] + `" data-position="left center"><img src="https://changsha.blob.core.windows.net/changsha/` + photoNames[i] + `" alt="" /></a>
							<h2>Submitted by ` + createdBys[i] + `</h2>
							<p>Date: ` + createdAts[i]  + `</p>
						</article>`;
					}

					$('#thumbnails').html(gallery);

					main.init();
				});
			}
  		}
	}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>

<style>
.g-signin-button {
  /* This is where you control how the button looks. Be creative! */
  display: inline-block;
  padding: 4px 8px;
  border-radius: 3px;
  background-color: #3c82f7;
  color: #fff;
  box-shadow: 0 3px 0 #0f69ff;
  cursor: pointer;
}
</style> 

<style lang="scss">
  .dropbox {
    outline: 2px dashed grey; /* the dash box */
    outline-offset: -10px;
    background: lightcyan;
    color: dimgray;
    padding: 10px 10px;
    min-height: 200px; /* minimum height */
    position: relative;
    cursor: pointer;
  }

  .input-file {
    opacity: 0; /* invisible but it's there! */
    width: 100%;
    height: 200px;
    position: absolute;
    cursor: pointer;
  }

  .dropbox:hover {
    background: lightblue; /* when mouse over to the drop zone, change color */
  }

  .dropbox p {
    font-size: 1.2em;
    text-align: center;
    padding: 50px 0;
  }
</style>