# blogs.core.ai

core.ai tech blogs

* [Important](#important)
* [Theme](#theme)
* [Creating a new blog](#creating-a-new-blog)

## Theme

* Theme used : https://themes.gohugo.io/themes/hugo-papermod/
* Visit https://adityatelange.github.io/hugo-PaperMod/ to know more about the theme.
* To update the theme, run:\
`git submodule update --remote --merge`
* To change configurations, go to `config.yml`
* To update default front matter, go to `archetypes/`

## Creating a new blog  

* Run `hugo new blogs/<blog-name>.md`
* A new md file will be created with the given name.
* Edit parameters in the front matter. Eg: `author`, `tags`, `description`
* Add your content to newly created md file.
# Important

* After cloning the repo, run:\
`git submodule update --init --recursive` \
  because submodules may not get cloned automatically.
* Before pushing your code into the repo, do the following for deploying the site:
  * Delete `docs` folder to remove the previous build.
  * Run `hugo` in terminal to create a new build.
  

  