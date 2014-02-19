Alloy REST API Adapter with built in caching support
=========================

A rest api adapter for appcelerator alloy that has built in caching

the caching code was refactored from this repo [https://gist.github.com/johnthethird/560183](https://gist.github.com/johnthethird/560183)


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
                type : "restapi_c",
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
    
If your response was cached content, the returned object will contain metadata about the cached data. The adapter will add a new `cacheInfo` object the JSON results that will look like this

    cacheInfo : {
        cached : true,
        cached_at : "2014-02-19 00:08:05"
    }

If you are interested in passing the cache information all the way through to your Alloy model, in most cases you would need to utilize the BackboneJS method on your object call `parse`; [see documentation here for additional information](http://backbonejs.org/#Model-parse). 

but your code in the Alloy model would look like this

    parse: function(_response) {
        var model = this;
        
        var models = [];
        
        // loop through all data in collection, create model and add to set
        models = _.map(_response.data, function(_i) {
            return new model(_i);
        });
        
        // save all of the pagination information for the model
        model.pagination = _response.pagination;
        
        // save any caching information for the model
        model.cacheInfo = _response.cacheInfo;
        
        // return the object to be returned collection
        return models;
    },
When processing a JSON response from the API that look like  this

    {
        "pagination": {
            "next_url": "https://api.instagram.com/v1/users/247944034/media/recent?max_id=fffff&client_id=XXX",
            "next_max_id": "645906744555277618_247944034"
        },
        "meta": {
            "code": 200
        },
        "data": [
            // INSTAGRAM JSON RESPONSE DATA - These will all be models added to the collection
        ],
        "cacheInfo" : {
            cached : true,
            cached_at : "2014-02-19 00:08:05"        
        }
    }


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

Events and Purging The Cache
-
The module responds to an application level event to purge the entire cache; `app.purge.cache` and it takes and optional parameter to display an alert with cache information when the purging is completed

    Ti.App.fireEvent('app.purge.cache', {showAlert: false})
    
Or you can call the purge cache method on the the model, any model that is created withe the adapter can purge the entire cache. A nice optimization would be to only purge a specific object type, but I just thought of that !!

    UserPhotos.save({
        type:'purge',
        onComplete: function(_response){
            Ti.API.debug( _response.itemsPurged + " Items were purged from cache");
        }
    });

when the purge is completed, an event is fired to indicate the process has completed `app.purged.cache` the event returns information on the number of items purged from the cache

    Ti.App.addEventListener('app.purged.cache', function(_event){
        Ti.API.debug( _event.itemsPurged + " Items were purged from cache");
    }

## License

(The MIT License)

Copyright (c) 2014 Aaron Saunders &lt;aaron@clearlyinnovative.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
