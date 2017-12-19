## branch for backup blog source

```shell
# clone the project
git clone git@github.com:xiejun901/xiejun901.github.io.git

# checkout to the backup branch hex
git checkout --track origin/hexo

# init the submodule, this is the fork of hexo theme: next
git submodule update --init --recursive

# install npm
npm install

# replace the marked.js to support math-jax better
mv marked.js node_modules/marked/lib/marked.js

# start the hexo blog !
hexo s --debug

```
