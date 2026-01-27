# Words and Code - A blog


To add a new post

```
hugo new post/name.md
```

To deploy changes

```
hugo
cd public && git add --all && git commit -m "Publishing to gh-pages" && cd ..
git push origin gh-pages
```

If the new post is not showing up, you may need to initialize the submodule theme:

```
git submodule update --init --recursive
```