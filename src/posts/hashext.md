---
title: Test Blog Post 6
date: 2024-03-04 11:00:00
tags:
 - test
 - idk
 - lots
 - of
 - tags
---

I am writing some stuff here

``` python
#!/usr/bin/env python3

"""
Usage: gen.py [--serve]

Options:
    -s --serve      Serve the built files.
    -h --help       Show this screen.
"""

from jinja2 import Environment, FileSystemLoader
from distutils.dir_util import copy_tree
import markdown
import frontmatter

import http.server
import socketserver
import os

from docopt import docopt

from dataclasses import dataclass

BUILD_DIR = 'build'
SRC_DIR = 'src'

class Page:
    def __init__(self, rel_path=None, out_path=None):
        self.rel_path = rel_path
        self.out_path = out_path

        if rel_path:
            self.data = frontmatter.load(os.path.join(SRC_DIR, rel_path)).to_dict()
            self.data["content"] = markdown.markdown(self.data["content"], extensions=["fenced_code"])
            if not out_path:
                self.out_path = rel_path.replace("md", "html")

        else:
            self.data = {}

        
def render_file(env, template, rel_filepath, **kwargs):
    full_filepath = os.path.join(SRC_DIR, rel_filepath)

    if os.path.isdir(full_filepath):
        print(f"Skipping {filename} as it is a directory.")
    if not full_filepath.endswith('.md'):
        print(f"Skipping {filename} as it is not a markdown file.")

    render(env, template, Page(rel_filepath), **kwargs)

def render(env, template, page=None, **kwargs):
    if page:
        out_filepath = os.path.join(BUILD_DIR, page.rel_path.replace("md", "html"))
    else:
        out_filepath = os.path.join(BUILD_DIR, template)
        page = Page()

    with open(out_filepath, 'w') as f:
        f.write(env.get_template(template).render(page=page.data, **kwargs))


def build():
    posts = []
    tags_dict = {}
    for filename in os.listdir(os.path.join(SRC_DIR, "posts")):
        rel_filepath = os.path.join("posts", filename)
        full_filepath = os.path.join(SRC_DIR, rel_filepath)

        page = Page(rel_filepath)
        posts.append(page)

        for tag in page.data.get("tags", []):
            tags_dict[tag] = None


    tags = list(tags_dict.keys())
    posts = sorted(posts, key=lambda x: x.data["date"], reverse=True )

    print(posts[0].rel_path)

    env = Environment(loader=FileSystemLoader("templates"))
    env.globals['posts'] = posts
    env.globals['tags'] = tags

    copy_tree('static', os.path.join(BUILD_DIR, 'static'))

    render_file(env, "layout.html", "index.md")
    render(env, "posts.html" )

    for post in posts:
        render(env, "layout.html", post)

def serve():
    PORT = 8000

    os.chdir(BUILD_DIR)
    try:
        with socketserver.TCPServer(("", PORT), http.server.SimpleHTTPRequestHandler) as httpd:
            print(f"Serving HTTP on 0.0.0.0 port {PORT} (http://0.0.0.0:{PORT}/) ...")
            httpd.serve_forever()
    except KeyboardInterrupt:
        print("Keyboard interrupt received. exiting.")
        httpd.shutdown()


if __name__ == "__main__":
    opts = docopt(__doc__)

    build()

    if opts['--serve']:
        serve()
```

