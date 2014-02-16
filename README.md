Alloy REST API Adapter with built in caching support
=========================

A rest api adapter for appcelerator alloy that has built in caching

the caching code was refactored from this repo [https://gist.github.com/johnthethird](https://gist.github.com/johnthethird)


The REST adapter is one of the most used adpaters on projects that we build here a Clearly Innovative. Also integrating third party APIs with request limits and just for overall performance benefits we felt the need to cache data at various times.

Also we believe that the rest api for alloy should stay true to the original backbonejs model so we have not made any modifications to the why the adapter functions. It has been written so your code that interacts with alloy models can be used in a javascript application with the default backbone sync adapter.

Enjoy, if you have issues, please log them

How it works
=======

Using the Cache
-

The settings can be added as configuration options when setting up you model. the adapter will use these values by default unless value is specified on the model or in the options on the sync adapter call

    /**
     * Model used to perform a global search on the API
     */

    exports.definition = {
        config : {
            adapter : {
                type : "restapi",
            },
            cacheSettings : {
                cacheSeconds : 3000,     // timeToLive default value
                pruneSeconds : 2520000,  // pruneSeconds default value anything this old is GONE!!
            }
        },
        //
        // THE MODEL
        extendModel : function(Model) {
            _.extend(Model.prototype, {
            });
            return Model;
        },
        //
        // THE COLLECTION
        extendCollection : function(Collection) {

            _.extend(Collection.prototype, {
            });
            return Collection;
        }
    };


you would setup the call like this when using it with your model

    var UserPhotos = Alloy.Collections.instance('UserPhotos');
    UserPhotos.fetch({
        timeToLive : 3000,    // how long in ms for item to persist in cache
        ignoreCache : false,  // can ignore cache completely and make the api call always
        success : function(_collection, _response) {
            Ti.API.info('OUTPUT');
            Ti.API.info(JSON.stringify(_collection.models, null, 2));
        },
        error : function(_error, _response) {
    
        }
    });

Using the Activity Window
-

you will need to provide a library for displaying loading activity; the library should support the following methods.

    /* how long to wait before closing window */
    activityWindow.setMaxTimeout();
    
    /* show the window, with a message */ 
    activityWindow.show([message to display]);
    
    /* hide the window */
    activityWindow.hide();

you would setup the call like this when using it with your model

    var UserPhotos = Alloy.Collections.instance('UserPhotos');
    UserPhotos.fetch({
        activityWindow : require('activityWindowLib'),
        activityWindowTimeout : 60000,
        success : function(_collection, _response) {
            Ti.API.info('OUTPUT');
            Ti.API.info(JSON.stringify(_collection.models, null, 2));
        },
        error : function(_error, _response) {
    
        }
    });
    
you could also leverage the flexibility of javascript and do this

    var UserPhotos = Alloy.Collections.instance('UserPhotos');
    
    /* set the properties on the collection object */
    UserPhotos.activityWindow = require('activityWindowLib');
    UserPhotos.activityWindowTimeout = 60000;
    
    /* fetch the objects for the collection */
    UserPhotos.fetch({
        success : function(_collection, _response) {
            Ti.API.info('OUTPUT');
            Ti.API.info(JSON.stringify(_collection.models, null, 2));
        },
        error : function(_error, _response) {
    
        }
    });
