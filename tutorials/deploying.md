# Deploying your own Chart Tool

Deploying your Chart Tool interface should be pretty straightforward, and can be done in about 15 minutes. You'll need:
- basic knowledge of how to connect to a remote server over `ssh` and copy files over using `scp`
- a server, preferably an up-to-date version of Ubuntu loaded with `node` >= 4.1.0 and `npm`
- access to a MongoDB instance, either on the same server or elsewhere
- optionally, your S3 bucket name, region, key and secret key, if you're using Chart Tool's [upload-to-S3](https://github.com/globeandmail/chart-tool/blob/master/tutorials/thumbnails.md) feature

For a few different reasons, we recommend you build the project locally before trying to deploy it to a server. It'll save you a lot of pain and server space.

----------


### **Step 1:** Build the interface
Locally, and in the root Chart Tool directory, run `gulp meteor-build`. This compiles down the Meteor-side of Chart Tool into a simple Node project that can be deployed anywhere.


### **Step 2:** Copy the built file to your server
Copy the meteor.tar.gz file to your server using [`scp`](http://www.hypexr.org/linux_scp_help.php) or something similar.


### **Step 3:** Unzip it
On your server, [untar](https://xkcd.com/1168/) the file and `cd` into the directory you've just created:
```sh
tar -xvzf meteor.tar.gz && cd bundle
```


### **Step 4:** Install npm dependencies
Meteor compiles down into a node project with a few npm dependencies, so let's install those:
```sh
cd programs/sever && npm install
```


### **Step 5:** Get wkhtmltopdf set up
Copy the link to the latest stable build of wkhtmltopdf from [this page](http://wkhtmltopdf.org/downloads.html). Note: do **not** use `apt-get` — you want the latest version of wkhtmltopdf with patched Qt, and the versions on `apt-get` tend to be out of date.

On your server, `wget` that link, then untar the file using `tar -xvf`. After that, `cd` into the wktmltopdf folder (it might be called "wkhtmltox") and move the binary to your `bin` folder:
```sh
wget <wkhtmltopdf file address>
tar -xvf <local path to file>
cd <folder generated by tar command>
cp bin/wkhtmltopdf /usr/bin/wkhtmltopdf # might have to run this as sudo
```
Make sure wkhtmltopdf is installed by running `wkhtmltopdf`. If you see some documentation and help options, you're good to go!

### **Step 6:** Set up a MongoDB instance
This part can be a bit tricky. You'll either want to set up a Mongo instance using a service such as [mLab](http://www.mlab.com) or run it on the same server. If you do decide to run it on the same server, follow [these instructions](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-16-04) and use `mongodb://127.0.0.1:27017` as your connection string.


### **Step 7:** Set your environment variables and run the app
You're almost there! All that's left is to set up our environment variables and run the application with some basic logging:
```sh
export PORT=80
export MONGO_URL='mongodb://<mongodb connection string>'
export ROOT_URL='<charttool root URL>'
node main.js >> /var/log/charttool.log 2>&1
```
To make things easier in the future and ensure your app restarts if a problem causes it to go offline, you might want to wrap the process above in an [Upstart script](https://www.digitalocean.com/community/tutorials/the-upstart-event-system-what-it-is-and-how-to-use-it).


### **Step 8 (optional):** Add your AWS keys
In addition to the environment variables above, if you're using the upload-to-S3 functionality in Chart Tool, you should add your AWS information as environment variables:
```sh
export S3_CHARTTOOL_BUCKET=<s3 bucket name>
export S3_CHARTTOOL_REGION=<region>
export S3_CHARTTOOL_KEY=<AWS key>
export S3_CHARTTOOL_SECRET=<AWS secret>
```
