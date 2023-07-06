# How to create this project

## Step 1:; Install jekyll

<https://jekyllrb.com/docs/installation/ubuntu/>

```bash
Install Ruby and other prerequisites:

sudo apt-get install ruby-full build-essential zlib1g-dev

Avoid installing RubyGems packages (called gems) as the root user. Instead, set up a gem installation directory for your user account. The following commands will add environment variables to your ~/.bashrc file to configure the gem installation path:

echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
Finally, install Jekyll and Bundler:

gem install jekyll bundler
```

## Step 2: Create an empty jekyll project

```bash
# In a repository we create the docs folder and install a new package
cd docs
jekyll new --skip-bundle .
bundle install

# Let's run the package to test it locally
bundle exec jekyll serve
```
