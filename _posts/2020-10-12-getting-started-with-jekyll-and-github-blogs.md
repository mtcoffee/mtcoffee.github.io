---
title: Getting Started with Jekyll and Github blogs
tags:
- jekyll
- github
toc_sticky: true
toc: true
comments: true
---

I was browsing around and saw people using github.io as a blogging url. Intrigued I looked into it and discovered github.io is not a bloging platform but does give you the capability to host static pages on github. To turn this in to blog you need something that will generate your blog into a set of static pages. Enter Jekyll. In this example I'm going to use a docker and a series of 1 liners to create a Jekyll blog on Github using the minimal mistakes theme developed by an indepdent dev.

# Prerequisites
To begin you will need
1. docker
2. git
3.  A github account with a [personal access token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token)

# Create a repo to hold the blog using the github API
```
#create repo using github API and change the name to your your github repo name
token="<your personal token>"
curl -H "Authorization: token $token" --data '{"name":"<yourrepo>.github.io","description":"A github pages blog"}' https://api.github.com/user/repos
```

# Create a local repo in docker to hold your blog
We will git clone the minimal-mistakes repo as a starting point for our blog

```
mkdir /etc/docker/jekyll
sudo chown 1000:1000 /etc/docker/jekyll
cd /etc/docker/jekyll
git clone https://github.com/mmistakes/minimal-mistakes.git .
```

# Create a docker container to serve content locally 
We will spin up a docker container to host the copy locally.

```
docker run \
  --volume="/etc/docker/jekyll:/srv/jekyll" \
  -e PUID=1000 \
  -e PGID=1000 \
  --restart=unless-stopped --name jekyll \
  -d -p 4000:4000 jekyll/jekyll:3.8 \
  jekyll serve --watch --force_polling --verbose
```

This will take a minute, you can run "docker logs jekyll --follow" to keep tabs on it.  Once its finished browse to http://dockerhost:4000

Once its up and running  some cleanup. 

```
docker exec -i jekyll sh -c 'rm .editorconfig .gitattributes CHANGELOG.md README.md screenshot-layouts.png screenshot.png; rm -r docs test .github'
```
# Add jekyll-admin and github pages
I like jekyll-admin as a local CMS and of course we will need the github pages plugin to push to github.

```
docker exec -i jekyll bash <<EOF
echo "" >> Gemfile
echo -e "gem 'jekyll-admin', group: :jekyll_plugins" >> Gemfile
echo -e "gem 'github-pages', group: :jekyll_plugins" >> Gemfile
exit
EOF
```
and now we'll restart the container
```
docker exec jekyll sh -c 'rm Gemfile.lock'
docker restart jekyll
```
Once its finished browse to http://dockerhost:4000/admin to see jekyll-admin

# Pointing to the Github Repo
```
docker exec -i jekyll bash <<'EOF'
git status
git add .
git config --global user.name <gitusername>
git config --global user.email <yourgithubregisteredemailaddress>
git remote set-url origin https://<gitusername>:<token>@github.com/<gitusername>/<yournewrepo>.git
#example:
#git remote set-url origin https://meatsac:<token>@github.com/meatsac/meatsac.github.io.git
exit
EOF
```
# To commit our blog to github
```
#to push to github
docker exec -i jekyll bash <<'EOF'
git add .
git commit -m "my new commit"
git push origin master
exit
EOF

#to pull current updates and then push new (useful if updating blog via other sources)
docker exec -i jekyll bash <<'EOF'
git pull
git add .
git commit -m "my new commit"
git push origin master
exit
EOF
```

# Finishing Up
At this point you now have a "hello world" blog hosted on github. To configure your blog you'll want to look at the config.yml file.  Have a look at these sources:
* [https://www.cross-validated.com/Personal-website-with-Minimal-Mistakes-Jekyll-Theme-HOWTO-Part-II/](https://www.cross-validated.com/Personal-website-with-Minimal-Mistakes-Jekyll-Theme-HOWTO-Part-II/)
* [Minimal Mistakes Official](https://mmistakes.github.io/minimal-mistakes/docs/configuration/)

To create a new post just head over to jekylladmin, create your post and commit using the commit block above. You may also prefer to use something like prose.io to manage your blog on line without requiring local resources.
