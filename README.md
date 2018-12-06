Posts for Hexo
---

### How to use

This repository is for my [blog](https://norang.io).
To render this posts properly, make simlink using commands below.

```
rm ~/blog/hexo/norang.io/source -rf
ln -sf ~/blog/source ~/blog/hexo/norang.io/source
```

### Tips for Hexo blogging

* to make "tags", "categories" or "about" menu:
  * create page named after that: `hexo new page tag`
  * edit `types` entry of index.md in the directory:
  ```
  title: All tags
  type: "tags"
  ```

* as 'theme-next/hexo-theme-next' doesn't parsing scheme config correctly, I just modified [this part](https://github.com/theme-next/hexo-theme-next/blob/b378211e6558aeea75e387a13fb9e6b28e99593c/source/css/main.styl#L5).
* 'theme-next/hexo-theme-next' doens't parsing font config correctly now.
