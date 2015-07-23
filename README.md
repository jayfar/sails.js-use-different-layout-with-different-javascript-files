# sails.js Use Different Layout Files with Different Javascript Files
This is an example of how to use different layout.ejs files with different javascript files included. Sails v0.11.0.

In my project, I wanted to take advantage of Sail's cool built-in ability to auto minimize/uglify javascript files for "sails lift --prod" in various layouts with different sets of javascript files.

In my project I had 2 different layouts -- layout.ejs and layoutadmin.ejs. I created a new /assets/jsadmin folder which holds my admin only javascript files. I left the sails existing /assets/js folder as-is to hold the javascript files for the public web page. My goal was for the /assets/js folder contents to be inserted between <!--SCRIPTS--> tags (sails does this by default) and the /assets/jsadmin folder contents to be inserted between the <!--SCRIPTS_ADMIN--> tags (I made up this tag name and I will add add support for this new tag below).


Each of these steps were checked in as a differenet set of commits in this git repository.

## Step 1
I created an empty sails.js v0.11.0 project using this command:

```
sails new sails.js-use-different-layout-with-different-javascript-files
```

## Step 2
I Added Admin and Home controllers along with admin and home views. I also added layoutadmin.ejs with <!--SCRIPTS_ADMIN--> tags that will be used in the next commit step.

I Added jquery js file and knockout js files that will be used in the next commit step.

The idea is for the js folder contents to be inserted between <!--SCRIPTS--> tags (sails does this by default) and the jsadmin folder contents to be inserted between the <!--SCRIPTS_ADMIN--> tags (will be done in the next couple of commits.


## Step 3
For development (sails lift), I modified so sails would populate the <!--SCRIPTS_ADMIN--> tags with the assets/jsadmin js files upon lifting.

I modified tasks/pipeline.js by adding a new variable called jsAdminFilesToInject which is very similar to the existing jsFilesToInject except it collects the js files from the jsAdmin folder. Note: I also had to export this new variable at the bottom of the pipeline.js file.

I modified tasks/config/sails-linker.js by adding a new devJsAdmin task where it looks for <!--SCRIPTS_ADMIN--> tags and calls the new .jsAdminFilesToInject added in the pipeline.js.

I Added a new task step to the tasks/register/linkAssets.js file which calls the devJsAdmin added above.

### Run sails in demo mode:
```
sails lift
```

Browse to http://localhost:1337/home - you will see it is using the layout template and viewing the source will show the following at the bottom (files pulled from js folder):
```
    <!--SCRIPTS-->
    <script src="/js/dependencies/sails.io.js"></script>
    <script src="/js/jquery-1.10.2.js"></script>
    <!--SCRIPTS END-->
```

Browse to http://localhost:1337/admin - you will see it is using the layoutadmin.ejs template and viewing the source will show the following at the bottom of the source (files pulled from jsAdmin folder):
```
    <!--SCRIPTS_ADMIN-->
    <script src="/jsAdmin/dependencies/jquery-1.10.2.js"></script>
    <script src="/jsAdmin/knockout-3.3.0.debug.js"></script>
    <!--SCRIPTS_ADMIN END-->
```

## Step 4
For production (sails lift --prod), I wanted to do the same as development except I first wanted to concat and uglify the production javascript that goes in my new <!--SCRIPTS_ADMIN--> tags.

I added a new jsAdmin section in the grunt tasks/config/concat.js file which pulls in the files from the previously added jsAdminFilesToInject in the pipeline.js to produce a concat/productionAdmin.js output file.

I added a new distAdmin section in the grunt uglify.js file which makes the concat/productionAdmin.js "ugly" by producing a new min/productionAdmin.min.js file.

I added a new prodJSAdmin section in the sails-linker.js file which adds the min/productionAdmin.min.js file between the <!--SCRIPTS_ADMIN--> tags.

Finally, I called this new prodJSAdmin from the prod grunt task by adding a line in the prod.js file.

### Run sails in production mode:
```
sails lift --prod
```

Browse to http://localhost:1337/home - you will see it is using the layout template and viewing the source will show the following at the bottom (using production.min.js):
```
    <!--SCRIPTS-->
    <script src="/min/production.min.js"></script>
    <!--SCRIPTS END-->
```

Browse to http://localhost:1337/admin - you will see it is using the layoutadmin.ejs template and viewing the source will show the following at the bottom of the source (using productionAdmin.min.js):
```
    <!--SCRIPTS_ADMIN-->
    <script src="/min/productionAdmin.min.js"></script>
    <!--SCRIPTS_ADMIN END-->
```
