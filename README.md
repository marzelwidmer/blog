# README

## Maintain

```bash
cd ~/git/blog/marzelwidmer.github.io
```

```bash
hugo -t github-style
```

```bash
cd public
```

```bash
git add . && git commit -m 'chore: publishing' && git push
```

[Creating a Blog with Hugo and Github in 10 minutes](https://www.youtube.com/watch?v=LIFvgrRxdt4)

[marzelwidmer.github.io](https://marzelwidmer.github.io)

```bash
git clone git@github.com:marzelwidmer/blog.git
```

```bash
cd blog
```

```bash
hugo new site marzelwidmer.github.io
```

```bash
cd marzelwidmer.github.io
```

```bash
cd themes
```

```bash
git clone https://github.com/niklasbuschmann/contrast-hugo.git
```

```bash
cd ..
```

```bash
vi hugo.toml
```

```bash
baseURL = 'https://marzelwidmer.github.io/'
languageCode = 'en-us'
title = 'Marcel Widmer'
theme = 'contrast-hugo'
```

```bash
hugo new posts/myposts.md
```

```bash
vi content/posts/myposts.md
```

```md
+++
title = 'Myposts'
date = 2024-02-17T12:42:12+01:00

# draft = true

+++

Test content.
```

```bash
hugo server
```

```bash
git submodule add -b main git@github.com:marzelwidmer/marzelwidmer.github.io.git public
```

```bash
hugo -t contrast-hugo
```

```bash
cd public/
```

```bash
git add . && git commit -m 'chore: test publishing' && git push
```
