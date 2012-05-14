backbone.configurator
=====================

Backbone.Configurator (Backbone.Config) is an extensible object-class that allows you to extract any and all hardcoded string dependancies from your Backbone classes and
manage them in a hierarchical object wrapped with the usual getter and setter functions, plus Backbone.Events, plus more.  The best analogy is with Backbone.Collection.
You use aa Collection object to manage a list of Model instances.  You use Backbone.Config to manage an object hierarchy that consists of all your application's config
information organized however you see fit.  

## Benefits

/* Copywrite 2012, David Shapiro - portions added to existing Backbone code*/

/* Subject to FFRUYL licensing - Feel free to rip and use as you like. */

/*globals Backbone:true, _:true, $:true*/

// @name: Configurator
//
// @tagline: Configurator for Backbone Apps- models, views and routers
//
// @description:
// Experimental code
// -Maintain Backbone MVC classes free of hardcoded dependancies including strings, css, text, and html fragments.
// -Create and modify application configuration object and inject into app bootstrap.
// -Modify configurations and trigger config:changed events during runtime to dynamically decorate your app.
// -View and Model classes can react to config events the same way views can react to data events
// -Configurator classes and config objects can be subclassed or mixed into to 
// -Class and inheritance structure follows the same conventions used in core Backbone modules EXCEPT
// for the wrapped config objects which follow more of a CSS type of property application from more general 
// to more specific

Backbone.Configurator = (function (Backbone, _, $) {
    "use strict";
    
    var Configurator = {}, config;
    Configurator.version = "0.01";
    
    //The general idea is to remove any and all hardcoded dependancies from your 
    //Backbone classes and list them here or add by subclassing, mixing in, or dynamically at runtime
    //changes to this object can trigger config change events that your application can react to 
    
    //sample, suggested skeletal configuration for basic Backbone modules
    //the idea is to use this as a filing system for dependancies in whatever logical
    //groupings make sense for your site
    //you can extend this object or set your own via
    //Backbone.Configurator.config = {your base config} prior to extending your own
    //Configurator classes
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
    
       
    Configurator = function(props, options) {
        this.config = $.extend(true,{}, this.constructor.config);
        this.add(props || {});
        this._configure(options || {});
        this.initialize.apply(this, arguments);
    };
    
    Configurator.config = config;
    
    // List of config instance options to be merged as properties. Empty by default.
    // Add any special instance options you want set directly on the config instance.
    var configOptions = [];

    // Set up all inheritable **Backbone.Configurator** properties and methods.
    _.extend(Configurator.prototype, Backbone.Events, {

        // recursively add or change properties to the config object 
        // see docs for jQuery.extend(deep:true) for behavior
        // triggers a change:config event unless {silent:true}
        add: function(props){
            if(typeof props == 'object'){
                $.extend(true, this.config, props);
                if(!arguments[1]){  
                    this.trigger('change:config', props);
                }
            }
            return this.config;
        },
        //alias for add
        set: function(props){
            return this.add(props);
        },
        
        // return a nested object member of the config obj for string 'key'
        // preforms an optional second level search and returns first match
        // override for different or additional fucntionality
        // returns false if no match
        // for the whole config obj use getConfig()
        get: function(key){
            if(this.config.hasOwnProperty(key))
                return this.config[key];
            else{
                for(var obj in this.config){
                    if(this.config[obj].hasOwnProperty(key))
                        return this.config[obj][key];
                }
            }
            return false;
        },
        // return the config obj
        getConfig: function(){
            return this.config;
        },
        //warning... this will replace the current config obj.
        //triggers a reset event unless silent:true
        resetConfig: function(conf){
            conf = conf || {};
            this.config = conf;
            if(!arguments[1]){  
                this.trigger('reset:config', conf);
            }
        },
           
        // Same functionality as Bacbone.View. To use, add values to the 
        // configOptions array. Included for general Backbone behavior compatability
        _configure: function(options) {
            if (this.options) 
                options = _.extend({}, this.options, options);
            for (var i = 0, l = configOptions.length; i < l; i++) {
                var attr = configOptions[i];
                if (options[attr]) 
                    this[attr] = options[attr];
            }
            this.options = options;
        },

        // Initialize is an empty function by default. Override it with your own
        // initialization logic.
        initialize: function(){}
    });
    
    // The self-propagating extend function that Backbone classes use.
    // Added  configProps to for deep copying
    Configurator.extend = function(configProps, protoProps, classProps) {
        var child = inherits(this, configProps, protoProps, classProps);
        child.extend = this.extend;
        return child;
    };
   
    // Shared empty constructor function to aid in prototype-chain creation.
    var ctor = function(){};

    // Helper function to correctly set up the prototype chain, for subclasses.
    // Similar to `goog.inherits`, but uses a hash of prototype properties and
    // class properties to be extended. 
    var inherits = function(parent, configProps, protoProps, staticProps) {
        var child;

        // The constructor function for the new subclass is either defined by you
        // (the "constructor" property in your `extend` definition), or defaulted
        // by us to simply call the parent's constructor.
        if (protoProps && protoProps.hasOwnProperty('constructor')) {
            child = protoProps.constructor;
        } else {
            child = function(){
                parent.apply(this, arguments);
            };
        }

        // Inherit class (static) properties from parent.
        _.extend(child, parent);
        
        // reset child.config to a deep copy since we want to remove any references
        // to parent.config before adding or changing config props
        child.config = $.extend(true, {}, parent.config);

        // Set the prototype chain to inherit from `parent`, without calling
        // `parent`'s constructor function.
        ctor.prototype = parent.prototype;
        child.prototype = new ctor();

        // Add deep copy of config properties (instance properties) to the subclass,
        // if supplied since we don't want to copy any nested references.
        if (configProps) $.extend(true, child.config, configProps);
        
        // Add prototype properties (instance properties) to the subclass,
        // if supplied.
        if (protoProps) _.extend(child.prototype, protoProps);

        // Add static properties to the constructor function, if supplied.
        if (staticProps) _.extend(child, staticProps);

        // Correctly set child's `prototype.constructor`.
        child.prototype.constructor = child;

        // Set a convenience property in case the parent's prototype is needed later.
        child.__super__ = parent.prototype;

        return child;
    };
    
    return Configurator;

}(Backbone, _, $));