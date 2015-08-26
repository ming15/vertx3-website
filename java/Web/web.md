Vert.x-Web is a set of building blocks for building web applications with Vert.x.

Think of it as a Swiss Army Knife for building modern, scalable, web apps.

Vert.x core provides a fairly low level set of functionality for handling HTTP, and for some applications that will be sufficient.

VVert.x-Web builds on Vert.x core to provide a richer set of functionality for building real web applications, more easily.

It¡¯s the successor to Yoke in Vert.x 2.x, and takes inspiration from projects such as Express in the Node.js world and Sinatra in the Ruby world.

Vert.x-Web is designed to be powerful, un-opionated and fully embeddable. You just use the parts you want and nothing more. Vert.x-Web is not a container.

You can use Vert.x-Web to create classic server-side web applications, RESTful web applications, 'real-time' (server push) web applications, or any other kind of web application you can think of. Vert.x-Web doesn¡¯t care. It¡¯s up to you to chose the type of app you prefer, not Vert.x-Web.

Vert.x-Web is a great fit for writing RESTful HTTP micro-services, but we don¡¯t force you to write apps like that.

Some of the key features of Vert.x-Web include:

Routing (based on method, path, etc)

Regular expression pattern matching for paths

Extraction of parameters from paths

Content negotiation

Request body handling

Body size limits

Cookie parsing and handling

Multipart forms

Multipart file uploads

Sub routers

Session support - both local (for sticky sessions) and clustered (for non sticky)

CORS (Cross Origin Resource Sharing) support

Error page handler

Basic Authentication

Redirect based authentication

Authorisation handlers

JWT based authorization

User/role/permission authorisation

Favicon handling

Template support for server side rendering, including support for the following template engines out of the box:

Handlebars

Jade,

MVEL

Thymeleaf

Response time handler

Static file serving, including caching logic and directory listing.

Request timeout support

SockJS support

Event-bus bridge

Most features in Vert.x-Web are implemented as handlers so you can always write your own. We envisage many more being written over time.

We¡¯ll discuss all these features in this manual.