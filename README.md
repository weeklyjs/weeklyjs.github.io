# weeklyjs.github.io

## Contributing

### Running locally

```sh
# fork and clone the repository
git clone git@github.com:<user>/weeklyjs.github.io.git

export JEKYLL_VERSION=3
docker run --rm -v "$PWD/docs:/srv/jekyll:Z" -p 4000:4000 -it jekyll/jekyll:$JEKYLL_VERSION jekyll serve
```
