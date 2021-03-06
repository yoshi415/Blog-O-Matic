[![npm](https://img.shields.io/npm/v/blog-o-matic.svg?maxAge=2592000)](https://www.npmjs.com/package/blog-o-matic)
[![Build Status](https://travis-ci.org/Blanket-Warriors/Blog-O-Matic.svg?branch=master)](https://travis-ci.org/Blanket-Warriors/Blog-O-Matic)
[![Dependency Status](https://david-dm.org/Blanket-Warriors/blog-o-matic.svg?style=flat)](https://david-dm.org/Blanket-Warriors/blog-o-matic)
[![devDependency Status](https://david-dm.org/Blanket-Warriors/blog-o-matic/dev-status.svg)](https://david-dm.org/Blanket-Warriors/blog-o-matic#info=devDependencies)

Blog-o-Matic
============
This is a node.js module that can pull down repositories from Git, and turn README's and metadata into Javascript objects.  This is intended to be used with a server that can listen for Github hooks, and pull down a blog hosted in a Git repository. There is an example of an expected blog format on: https://github.com/Blanket-Warriors/Blog.

Example Usage:
```js
import Blog from "blog-o-matic";
import path from "path";
const blogRepository = "https://github.com/Blanket-Warriors/Blog";
const blogOptions = {
  branch: "myBranch",
  assetsUrl: "/blog-assets"
};
const blogDirectory = path.resolve(__dirname, "../../public/assets");

// Initialize the blog
const blog = new Blog(blogRepository, blogDirectory, blogOptions);

// Update my blog
myBlog.update().then(function(posts) {
  console.log(posts);
});
```

Updating the blog will clone the repo if it hasn't been updated before, and will reset the repo to the current remote branch's head if it already exists.

After it has been cloned down, we can feel free to access properties and various methods:
```js
myBlog.get();
myBlog.get("my-blog-post");
myBlog.destroy();
```

Integrating with an Express-served website
------------------------------------------
This module was made to be used with Express-served websites (we use this with the [Blanket Warriors homepage](http://www.blanketwarriors.com/blog)). It is very important to us that this is simple to use in this way.

An example of a server, serving a simple blog:
```js
import express from "express";
import Blog from "blog-o-matic";
import path from "path";

const server = express();
const blogRepository = "https://github.com/Blanket-Warriors/Blog";
const blogOptions = {
  branch: "2.0",
  assetsUrl: "/blog-assets"
};

// Initialize the blog
const blog = new Blog(blogRepository, path.resolve(__dirname, "../../public/assets"), blogOptions);
blog.update()
  .then(function(posts) {
    console.log("Updated!", Object.keys(posts));
  }, function(error) {
    console.log("error!", error);
  });

// Serve the blog
server.use("/blog/update", () => blog.update());
server.use("/blog-assets", express.static(blog.postsDirectory));
server.use("/blog", function(request, response) {
  const alias = request.url.replace(/\//g, "");
  const blogPost = blog.get(alias);
  if(blogPost) {
    response.send(blogPost.post);
  } else {
    response.send(Object.keys(blog.get()));
  }
});

let port = process.env.PORT || 2000;
server.listen(port);
console.log("listening on port", port);

```
