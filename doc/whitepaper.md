# Sharing and using modular user interface for websites

Web development is evolving at a fast pace. Yesterday websites had lots of HTML pages and was powered by monolithic infrastructures, today it is the age of website as application. Monolithic websites is still a thing but it is more and more common to see kind of modular websites with client side frameworks or libraries such as Angular or React.

This tendency offers people who wants to build websites different choices. However, it requires to learn a new way of building websites and new patterns or even paradigms. On top of that it often constrains them to use optional extensions, plugins or libraries that must work well with the website stack.

On the other side, there is another tendency that we commonly see in landing pages : it's templates. Template is a great thing for setting up websites in seconds. But the more we use templates, the more websites will look like each other which is not really expected. The purpose of a landing page is firstly to tease a solution and secondly, and not the least, appealing visitors with a strong branding and personality.

This paper is about a solution that aims to be simple. It follows the tendencies detailed above and it offers an alternative to enhance websites user interface (UI) without worrying about website stack compatibility.

## Webcomponents

Webcomponents [[1]](#wb) is a well-formed way to build websites with pieces of UI. This is an open source project being added by the W3C to the HTML and DOM specifications. The mission is to allow developers to build structured web applications with standard features that allow to make and use UI components (kind of widgets). Webcomponents [[1]](#wb) released 4 main features :
- HTML templates : describes a method for defining static DOM subtrees to make the document manipulable.
- HTML imports : feature for including HTML pages in another HTML page. It is important to note that Mozilla Firefox won’t support this feature, it may be deprecated.
- Custom Elements : describes a method for creating our own DOM element.
- ShadowDOM : describes a method for encapsulating DOM trees in nodes. It allows to isolate the shadowed document from the parent document.

Below are currently (10-20-2017) browsers support for the 4 features :

|                  | Chrome | Opera  | Firefox    | Safari | IE/Edge |
|------------------|--------|--------|------------|--------|---------|
| HTML Templates   | Stable | Stable | Stable     | Stable | Stable  |
| HTML Imports     | Stable | Stable | Abandoned  | Not yet| Not yet |
| Custom Elements  | Stable | Stable | Flag       | Nightly| Not     |
| ShadowDOM        | Stable | Stable | Flag       | Stable | Not     |

## Creation

A creation is a set of HTML, JavaScript and CSS which mixed together result in a piece of UI. The purpose of a Wooble creation is to be easy to be crafted and easy to be tested, thus anyone knowing those 3 basic stacks is able to make and to publish his creation. A published creation can be used by anyone in any website.

A creation is versioned to keep all changes.

Because Wooble rely on JavaScript in order to be able to use a creation, a creation requires a mandatory piece of code in JavaScript, it is its interface and it looks like the following.

```javascript
// Creation's base code in JavaScript
class Woobly {

	constructor(params) {
		this.document = document.body.shadowRoot;
	}
}
```

Wooble aims to be compatible with most browsers. A creation as to be “babelified” [[2]](#babel) (using BabelJS for transpiling JavaScript code to ES2015 standards).

The constructor is the creation initializer, when a user uses it and calls it, it will call the creation’s constructor. The creator is free to create and use any attributes within the class.

A creation uses a builtin parameters system by providing the parameter `params` in the constructor. This is the creation input and its type is `Object`. It is an handy feature for the user allowing to call a creation with whatever parameters he needs, making any creation customizable. The creator is free to define what is customizable. Here is an example of a creation using a custom text as parameter.

```JavaScript
// A creation's script code
class Woobly {

	constructor(params) {
		this.document = document.body.shadowRoot;

		// Prints out the parameter “customText”
		console.log(params.customText);
	}
}
```

Thus, `params` is similar to a bundle of variables making it a handy feature for customizing any creation.

A creation is a piece of UI, a user would likely need to control it if it’s dynamic. Let’s say a dropdown menu : it has to hide or show a menu when needed. A creation is based on an interface, as for any interface, it is easy to add new methods, such as the following.

```JavaScript
// Creation : dropdown menu
class Woobly {

	constructor(params) {
		this.document = document.body.shadowRoot;
	}

	hide() {
		// Code that hides the dropdown menu
	}

	show() {
		// Code that shows the dropdown menu
	}
}
```

To be implemented in a website a creation uses ShadowDOM feature of Webcomponents [[1]](#wb). ShadowDOM allows us to encapsulate a creation in any website without worrying too much about the DOM structure in which a creation is used. Considering this usage, a creation has two document APIs which are :
- The creation’s document : it is the shadowRoot,
- The website document : it is the root document where the creation is implemented.

The creator is able to call the two documents.

```JavaScript
// A creation
class Woobly {

	constructor(params) {
		this.document = document.body.shadowRoot;

		// Prints out the creation’s document
		console.log(this.document);

		// Prints out the root document
		console.log(document);
	}
}
```

The creation’s document is a shadow root, it does not provide certain methods like `createElement`, the creator may call `document` to use those methods. Providing the root document is mandatory because the creator will probably need to select the creation’s elements for customizing or for simply querying elements. The root document and the shadow root document is like two realms which do not communicate.

## Package and build

Package is a end user feature. It allows anyone to group some creations in one place : similar to the way we use folders on a computer. Packages are not only here to put creations in some kind of folders, it has features that read all the creations to compile them in one library : this action is called “building a package”.

In order to compile all creations within a package, Wooble parses the three files of each creation (as a reminder a creation is composed of a JavaScript file, a HTML file and a CSS file which is optional). As a result, a package is compiled into one single JavaScript file and wrapped with the Wooble API, this file is the representation of the package which embed all creations.

A creation package has a unique alias within a package to avoid conflict with other creations. Moreover, it is possible and highly encouraged to setup the default parameters for each creations which offers the user the choice to set his own parameters or to use the default ones.

## Usage

To call a creation, the user may need to use its alias :
```JavaScript
Wb(‘myCreation’)...
```

`Wb` is the Wooble API which includes the user package and `myCreation` is a creation with the alias “myCreation”.

Wooble API provides some methods in order to manipulate the creations :
```JavaScript
Wb(‘myCreation’).init(‘#divid’)
```

`init` allows the user to call and to put the creation in a specified element, also called the “target”, it can be either the id attribute or the class attribute. If the user sets a class as a target which selects multiple elements, Wooble puts the creation in all selected elements. `init` also returns an array of objects which are all instances of the creation (one per element selected with the target).

```JavaScript
Wb(‘myCreation’).init(‘#divid’).then(objs => {
	console.log(objs[0]);
});
```

Thanks to this promise call, the user is free to manipulate a creation after its initialization. As a reminder, a creation provides custom methods defined by the creator, those methods can be used right after the initialization.

`init` is able to read optional parameters if the user don’t want the default ones

```JavaScript
var params = {
	text: ‘hello Wooble’
}
Wb(‘myCreation’).init(‘#divid’, params)...
```

`get` is a method for retrieving instantiated creations with the given alias. The method is useful to get the creation instances after using `init`.

```JavaScript
Wb(‘myCreation’).get()
```

`destroy` is a method for removing instantiated creations with the given alias from memory.

```JavaScript
Wb(‘myCreation’).destroy()
```

`refresh` is a method for refreshing instantiated creations with the given alias. If the user specified his own parameters, it will refresh with those parameters otherwise it’ll take default parameters of the creation.

```JavaScript
Wb(‘myCreation’).refresh()
```

`callMethod` is a method for calling a given method from instantiated creations with the given alias. It’s a handy method that allows to “bulk” call instantiated creations method.

```JavaScript
Wb(‘myCreation’).callMethod(‘methodName’, parameters…)
```

`setAttribute` is a method for bulk updating a given attribute from instantiated creations with the given alias.

```JavaScript
Wb(‘myCreation’).setAttribute(‘attributeName’, value)
```

## Wooble platform

The platform is the front-end for everything detailed earlier. Creators must have some incentives for creating and publishing their idea. The platform can be used as a portfolio. In fact, a creator will publish what he crafts and everything will be listed to users who could be interested. The more a creation is successful, the more a creator will be recognized for his work.

The platform provides an editor with a embed preview to write creations so a creator is able to craft a creation without bothering to create his files and to test the creation on his side. It makes sure it's autosaved, nothing can be lost.

Users can manage their package through the platform. Wooble provides a content delivery network (CDN) URL for each package, it allows to use the packages with a simple `script` tag in HTML. Using CDN is to make sure the user experience is optimal (it loads a package faster). Plus, it is possible to setup default parameters for each creation and seeing results instantaneously so that the user know how it will render in his website.

## Roadmap

- [x] Write the script that builds and manage packages with the native JavaScript - Write Wooble API in native javascript (ES2015)
- [x] Write a REST API
	- [x] Manage creations
	- [x] Manage packages
- [x] Make a frontend client to create, to manage and to use creations through an interface
- [ ] Write a Wooble API for most frameworks and libraries
	- [ ] Angular
	- [ ] ReactJS
	- [ ] VueJS
	- [ ] EmberJS
- [ ] Write a Wooble command line interface for most package manager/dependency manager
	- [ ] Composer / Artisan
	- [ ] NPM
	- [ ] Rake
- [ ] Write plugins of Wooble for most CMS
	- [ ] Wordpress
	- [ ] Jekyll
	- [ ] TextPattern

## Conclusion
Wooble provides a way to share and use piece of UI efficiently. Each creation considers W3C standards, accessibility and compatibility. Creators are for the most part professional web designers and UX experts so they know how to make efficient UI.

Users don’t have to worry if the creations they use will work. They are free to use any creation wherever they want in their websites. Moreover, whatever the stack a user has, say a static website or a website powered by a huge monolith framework, he can be sure Wooble provides what he needs without changing anything from his current stack.

Wooble also provides a nice way for integrated web solutions (such as Stripe or MailChimp) to help their users to customize them at needs. Brands often has to make a whole interface to help the users to customize their solution, it often results in friction that can be avoided by using Wooble.

## References

[1] <a name="wb"></a> https://www.webcomponents.org

[2] <a name="babel"></a> https://babeljs.io/
