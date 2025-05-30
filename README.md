# How to build a blog using Material for MkDocs

## Installation

### Create a virtual environment

```sh
python3 -m venv venv
```
### Activate the virtual environment

```sh
source venv/bin/activate
```
### Install Material for MkDocs

```sh
pip install mkdocs-material
pip install -r requirements.txt
```

## Create a new site

```sh
mkdocs new .
```

## Serve the site during development

```sh
mkdocs serve
```

## RSS Feed

look at http://localhost:8000/feed_rss_created.xml

## Deploy the site

```sh
mkdocs build
rsync -hrvz --delete -e "ssh -i ~/.ssh/id_ed25519_key" site/* <user>@<host>:/usr/local/www/
```
