backbone.configurator
=====================

/* (C) 2012, David Shapiro - portions added to existing Backbone code */

Backbone.Configurator (Backbone.Config) is an extensible object-class that allows you to manage your Backbone.js configuration(s) by moving any and all hardcoded string dependancies from your Backbone classes and
manage them in a hierarchical object wrapped with the usual getter and setter functions, plus Backbone.Events, plus more.  The best analogy is with Backbone.Collection.
You use a Collection object to manage a list of Model instances.  You use Backbone.Config to manage an object hierarchy that consists of all your application's config
information organized however you see fit.  The ideal use-case is in any situation where your Backbone object-classes need the flexibility to go beyond a single page app
and mutate to handle different presentation contexts, data sets, etc. It is designed to have the minimalist functionality
of typical configuration classes seen in server-side MVC frameworks with Backbone.Events and Backbone inheritance style
mixed in.

If your Backbone classes need the flexibility to handle more than one presentation context, your should be using something to abstract and manage the context dependancies.
Afterall, embedded string dependancies in your Backbone.js code such as templateId:"theIDinMyHTML" or className:"cssClassString"
is tantamount to embedding inline event handlers in your html: <a onclick="alert('clicked');return false">

## Benefits

* Maintain Backbone MVC classes free of hardcoded dependancies including strings, css, text, html fragments, routes, mappings, urls, jQuery selectors, class names, switches, 
default data attributes, etc. 
* Application configurations can be extended, instantiated, modified or reset at runtime. Instantiate your config object and inject into your app.
* Prototype inheritance, and object instantiation is the same style as Backbone.js except for the managed Config object which is not overridden, but  
jQuery deep-extended along the inheritance chain allowing for cascading configurations.
* Modify configurations and trigger config:changed events during runtime to dynamically decorate your app. Your other Backbone objects (views, models, routers)
can listen for config events and react accordingly. I.E. View.on(config:change, @render) -> Config.set({templateId:'#newTemplate'}) -> triggers config:change
* Ideal for situations where your Backbone views (controllers) and routers need the flexibility to handle different templating and rendering situations
depending on different display contexts.
* You can override the utility functions (or add to) with those more suited to your needs. As with the rest of Backbone, the functionality is the minimal necessary.
* A suggested, skeletal config object is included which you can extend or overwrite with your own.

## API: ##

**Config.extend({settings}, {protoProps, {classProps})**
subclass a Config with optional settings object included before the usual Backbone.extend options

    var SubConfig = Config.extend({
        test:{
            foo:'bar'
        }
    });

**new Config({settings}, {options:{fresh:true|false}})**
instantiate a new config instance with optional settings object listed before the Backbone options object. 
Include {fresh:true} in the options to disclude any inherited config settings.

    var myConf = new SubConfig({
        test:{
            bar:'baz'
        }
    })

    myConf.config = {
        test:{
            foo:'bar',
            bar:'baz'
        }
    }

**.get('prop', [true])**
return the value for a property string. This implementation does a breadth search of the first 2 levels only for minimalist performance 
reasons. Including true as the second arg will wrap the returned value in a new Config instance. This can be overridden for more complex searching, hashing, 
flattening, etc. 

    myConf.get('foo') // 'bar'
    
    myConf.get('test', true).get('bar') // 'baz'

**.set({settings}, [loud:true], 'event_string']])**
deep copies new settings into the Config instance. If loud:true, a generic change:config event is triggered. 
An optional custom event string can be passed in.   Other Backbone objects can bind listeners in the usual way.

   myConf.set({
        test:{
            another:{propHash}
        }
    })

    myConf.get('test') // {
                            foo:'bar',
                            bar:'baz',
                            another:{propHash}
                          }

**.has('property')**
determine if property exists (not yet implemented)

    myApp = new Backbone.Router(myConf);

Sample, suggested skeletal configuration for basic Backbone modules. The idea is to use this as a filing system for dependancies
in whatever logical groupings make sense for your site.
You can extend this object or set your own via Backbone.Configurator.config = {your base config} prior to extending your own
Configurator classes.
    
    config = {
        //
        app: {
            bootStrapData: null,
            pager:false,
            filter:false,
            //css strings
            css: {},
            // jQuery selector strings
            selectors: {
                domAttachClass: '', 
                rootPageElem: 'body'
            },
            // avoid confusion with css classnames
            klassNames: {} 
        },
        router: {
            routes:{
                '': 'index'
            },
            //reverse hash of routes for use with router.navigate()
            paths:{
                index: ''
            }
        },
        item: {
            urlRoot: '/'
        },
        itemView: {
            tagName: 'article',
            className: null,
            id: null,
            templateId: null,
            css: {},
            // event key strings for your events hash
            events: {},
            // html strings too short to be worth templating
            html: {},
            selectors: {},
            // text strings
            text: {}
        },
        collection: {
            url: '/'
        },
        collectionView: {
            tagName: 'section',
            className: null,
            id: null,
            templateId: null,
            itemView: null,
            css: {},
            events: {},
            html: {},
            selectors: {},
            text: {}
        },
        history: {
            root: '/',
            pushState: true
        }
    };
    
    Configurator.config = config;

## Example usage coming soon