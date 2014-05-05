# Discourse Patches for Handlebar Injectors

This patchset adds a facility to Discourse Handlebars parser to apply patches injected by plugins.

## Description

This patchset wraps the handlebars.parse function to overwrite the functionality. When called it first parses using the original method and then applies previously added patches to the resulting Abstract Syntax Tree object returned by handlebars.parse (in the order provided).

For matching the wrapper uses an MD5-Hash of the source-string. Thus allowing fine-grained control without the necessacity of knowing the filename while at the same time adding simple version control. As such you need to register your patch-function against an MD5-Hash of the Source String. See the section below to learn about how to do that.

## API Description

### register_assets :template_injector

This patchset adds a new option to the `register_asset` function for plugins: `:template_injector`. If provided these assets will be included at the end of vendor.js before discourse is loaded. This is necessary as templates are rendered at the moment they are included and therefore all patches need to be registered there.

Example:

    register_asset "javascripts/inject_tag_templates.js.erb", :template_injector

### TemplatePatcher

At the time of including these patchers, the `window.TemplatePatcher` object is already present and accepting patchers.

#### addGeneralPatcher

Parameters:

 - md5hash
 - callback-function


Allows to register a new patcher against a given md5hash. This hash might also be a list of hashes the function will be registered for.

When the md5hash matches, the function will be called with the following parameters:

 - the current AST (Abstract-Syntax-Tree-Object)
 - the hash
 - the source string

The function is expected to change the AST in place. If the function returns something, this will be taken as the AST-object from that point on and replace the existing one.

Example:

  window.TemplatePatcher.addGeneralPatcher([
    "3fe4d7d24e07e91d38b67ffde45780f5",
    "f5a96e884170ace97506d3e4ca3eb3d3" //  at 0f62a6f132
    ],
    function(ast, hash, str){
        var tag_inject = <%= evaluate("./inject/composer_tagging.html").to_json(); %>;
        window.TemplatePatcher.insertAt(ast, 0, tag_inject);
    });

#### insertAt

The TemplatePatcher has a handy method to traverse the AST-Object and change its content in place. `insertAt` takes the following parameters:

 - the AST object
 - the _path_
 - the code to inject
 - options

The AST object usually being what is passed to the function, while the path may either be an integer marking the position in the highest level of the AST to be put at or a string in HAQL-Patch-Format to identify the position in the tree. The code may either be an AST-Object or a template-string to be passed to the parse function (which will itself be running through the patcher, too).

The following non mandatory options are known:

 - shift
 - replace

If `shift` is found the insertion will not be done in place of the path but moved by the given shift of positions. This can be very useful to for example put an item _behind_ an item found with the path (use shift: 1 then).

`replace` indicates that the original item shall be removed from the AST after the new one(s) have been inserted. Great for replacing/wrapping objects yourself and avoid duplication.

#### replaceWith

Alias for insertAt with options.replace = true.

#### setDebug

Set the debug mode to true or false. Very helpful in development.

#### setFinder

Helpful to find the right md5hash in development. Pass a string here and if debug is set the parser will `console.log` each occurance in strings passed as well as the corresponding hash.

## Understanding HAQL/AST

### Basics
The Handlebars-Abstract-syntax-tree-Query-Language is a text-based, space-separated query-language in the spirit of JQuery for the intermediate Syntax-Tree-Objects Handlebars builds internally. To understand HAQL, one first need to understand the basics of AST.

AST basically parses a Tree-System of `{{`-Handlebar-tokens. These might be nested (like in the case of `for`, `if` or `view`). Once parsed AST stores them as an Array of AST-Objects in the `statements`-attribute. As said, they can be nested so a given object in the Array can have a `statements` attribute itself. This is indicated by having a `mustache` attribute with meta information. One meta-information in there is the `id.string` property, which indicates which kind of nesting this is (like `if`).

In some cases, there is a double nesting depending on a condition, like in the case of an `else` in `if` or `else` for `each`. This block is stored in the `inverse.statements` of the object. Now that we've understood this basic concept, let's take a look at our first HAQL.

### Getting into HAQL

HAQL in its simplest form tries to match an element against the `id.string` of the current statements-list. Like the HAQL string `"if"` would match the first occurance of the `if`-block, the `"each"` the corresponding "each" respectively. Now, in order to get into the nested element you can add another matcher with a space. so `"if if"` would match the first if inside the first if, while `"if each if"` would match the first if in the first each in the first if. Simple as that.

### Indexes with HAQL
But sometimes you want to match something else then the _first_ occurance. For that HAQL knows about indexers, which are enclosed right on the matcher within square brakets like this `"[5]"`. Allowing to make little more complex queries matching not the first occurance – attention, this is a 0-based index – so `"if[2] each if[1]"` would match the second if inside the first each of the third if.

You can use the same technique to match non-mustache elements, just omit the matcher and it works as a simple index. `if[2] each [3]` will match the third element in the statements-list of each in the third if.

### Reverse-Blocks
In some cases you don't want the if but the else block. Unfortunately this isn't very clear from just looking at the statement (it could belong to if or each) so the syntax for that is a prefixed with its counter part. `"if[1] if-else each each-else [3]"` will match the third element of the else block of an each inside an each inside the the first else of the second if.

Essentially, that is it. Start playing or check out the tagger plugin for more help on this.

## For Development

There are two handy methods to help one when developing using the HAQL-Methods, making it easier to find the right point for matching: `setDebug` and `setFinder`.

If you want to develop a patch for an existing template, first register a new JS file with the `:template_injector` option and restart the server:

    register_asset "javascripts/my_injector.js.erb", :template_injector

In that file first set Debug mode and tell templatePatcher to inform you about all occurances of a very specific part of the original template. In this case we are looking for the class-name "topic-title":

    var patcher = window.TemplatePatcher;
    patcher.setDebug(True);
    patcher.setFinder("topic-title");

Now, after reloading, opening the browser console, it shows all occurances of that string in all templates. Copy the corresponding hash (`061dd3942f735486fbc91b5c7dfcf7a6` in this case) of the template you want to change and attach a debugger into the callback for callback. Now reload the page with the console open:

    patcher.addGeneralPatcher("061dd3942f735486fbc91b5c7dfcf7a6",
        function(ast, md5hash, string){
            debugger;
        });

Now you'd have access to the ast-object from within your debugger, can introspect it and write the corresponding HAQL-code and replace it with the debugger in your file. And you are done!