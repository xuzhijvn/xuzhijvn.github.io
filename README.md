# Tony Blog

创建：

```sh
hugo new -c content/zh-cn posts/newpost.md
```

编译：

```sh
hugo -d docs
```

主题升级：

[razonyang/hugo-theme-bootstrap](razonyang/hugo-theme-bootstrap)

```sh
$ cd themes/hugo-theme-bootstrap
$ git fetch
$ git checkout [version]
$ cd ../../
$ git add themes/hugo-theme-bootstrap
$ git commit -m 'Upgrade the theme'
```

- Replace the `[version]` with the latest version. The version can be listed by `git tag -l | sort -rV`.
- You can also checkout the `master` branch for getting the latest commit.
