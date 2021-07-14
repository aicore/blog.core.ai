---
title: "Hugo and Svelte"
date: 2021-07-07
weight: 1
aliases: [""]
tags: ["hugo", "svelte"]
author: ["Dhruv Eldho Peter", "Tapan Manu"]
categories: ["comparison"]
series: [""]
description: "A comparison between Hugo and Svelte"
summary: "A comparison between Hugo and Svelte"
searchHidden: false
# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

# Svelte

Svelte is one of the new approaches to build and test new frameworks. Traditional frameworks like React and Vue do their work in the browser while Svelte shifts that work into a compile step that happens when you build your app.

## Advantages

* Svelte writes code that surgically updates the DOM when the state of the app changes. 
* Web apps are fast in rendering components, compared to native react apps.
* Directly interacts with HTML-CSS components. 
* Svelte converts your app into ideal JavaScript at build time, rather than interpreting your application code at run time. Hence, performance is much better.
* Inherits features like state,props in react.

## Disadvantages

* Any bugs in the code would not let the app crash down, but does not render the entire component. 
* Prerequisite knowledge on ReactJs is needed for efficient app building.

# Hugo

Hugo is a static site generator. Unlike systems that dynamically build a page with each visitor request, Hugo builds pages when you create or update your content.
Hugo takes a source directory of files and templates and uses these as input to create a complete website.
Hugo is for people who want to hand code their own website without worrying about setting up complicated runtimes, dependencies and databases.

## Advantages

* Extremely fast build times
* Many themes are available online for free. So creating a site for a beginner will be easy.
* Content files and template files are stored separate. So if a person doesn't want to get into designing the site, he can just change the content without going into html codes.
* Content to be displayed is written in markdown language. So even if the user doesn’t know HTML, CSS or JS, he can create a simple site using the themes.
* Content files can also support HTML code, so designing can even be done through it.
* Users can use shortcodes, which are simple snippets inside your content files calling built-in or custom templates. They are used to avoid writing raw html code in md files. Shortcodes are reusable and can make coding easier for beginners.
* Supports HTML,CSS and JS.
* Documentation is good.

## Disadvantages

* Need to study lots of variables in Hugo to create complex sites.
* By default, the same structure that works to organize your source content is used to organize the rendered site.
* Passing data between pages is difficult.
* Hugo is good only for static data.

# Comparison


| Metrics | Hugo | Svelte |
| --- | --- | --- |
| Build time | It is extremely fast | Faster than native react apps. |
| Prerequisites | HTML,JS,CSS | Prerequisite knowledge in ReactJS is required. Syntax is similar to Vuejs. |
| Beginner-friendliness | It is easy for the user if only content manipulation is needed. To create templates and layouts, they need to study about Hugo variables and should understand the basic workflow. | Not as friendly as beginners need to have an idea on ReactJS, but this is comparatively easier to use than React. |
| Data passing | Passing data between pages is difficult. | Props can be used to pass data between components. |
| Supports built in themes | Lots of themes are available in Hugo. | No built in theme support.Npm supports react styled components only. |
| Website Complexity | Best for creating simple static websites. Eg: blogs | Websites are dynamic. Routing could be added via react router. |
| Development community | Hugo has good development community support. | Svelte is a relatively new framework. Hence community support is only developing. |
| Installation and setup | Easy and fast to install and setup. | Easier to install via npm. |
| Plugin Support | Hugo doesn't have plugin support. | Svelte supports plugins via npm. |
| Template Syntax | It's fairly non-standard and confusing for beginners. | It can be modified easily for a developer. Depends mainly on Javascript and ReactJS components. |
| Content-Driven | Hugo is great for content-driven websites. | Less content driven support. |
| Data files supported | Doesn't support XML as a data file type. However, YAML, JSON, and CSV are supported. | JSON and CSV file support |
| Content files supported | Html and markdown | Only html is supported |
| Debugging | Debugging is not easy  | Not easy in svelte. Errors are left undetected. |
| Size | Hugo is light | Svelte app size is comparatively larger, since node modules are incorporated.(Developer's perspective). |