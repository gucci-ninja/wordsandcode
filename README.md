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