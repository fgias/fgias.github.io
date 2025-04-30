# Minimal Mistakes remote theme starter

Click [**Use this template**](https://github.com/mmistakes/mm-github-pages-starter/generate) button above for the quickest method of getting started with the [Minimal Mistakes Jekyll theme](https://github.com/mmistakes/minimal-mistakes).

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew update
brew install ruby
echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
brew install rbenv ruby-build
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zprofile
echo 'eval "$(rbenv init -)"' >> ~/.zprofile 
rbenv install 3.2.3
rbenv global 3.2.3
rbenv rehash
sudo gem install bundler jekyll
bundle install
cd ./fgias.github.io
bundle exec jekyll build
bundle exec jekyll serve
```