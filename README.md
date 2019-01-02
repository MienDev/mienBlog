# About

this blog is built with [hugo](https://gohugo.io/), and based on the theme [mainroad](https://themes.gohugo.io/theme/mainroad/)

```
hugo new site mienBlog
cd themes && git submodule add https://github.com/MienDev/Mainroad-hugo-theme.git mainroad

cp -R mainroad/exampleSite/*.* ../

cd .. && hugo server -D
```