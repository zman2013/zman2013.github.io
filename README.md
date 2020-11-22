# command 
## add page
`npx hexo new <title>`

## Cannot GET /categories/ , Cannot GET /tags/
```shell
hexo new page categories
# type: "categories" in front-matter
hexo new page tags
# type: "tags" in front-matter
```

Link [/hexo-theme-next/issues/325](https://github.com/theme-next/hexo-theme-next/issues/325)


## Markdown原生写法嵌入图片
```
_config.yml
post_asset_folder: true
```

通过命令`hexo new title`建立的页面会自动建立对应的图片目录(名称与title一致)，将图片放在下面就可以直接引用。如果已存在的页面，需要手动建立对应的文件夹。

Link [Assert forlders](https://hexo.io/docs/asset-folders.html)

