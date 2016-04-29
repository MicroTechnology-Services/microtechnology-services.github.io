MTSI Developers Blog
---
Available at https://microtechnology-services.github.io/

Editing the blog
---

    git clone git@github.com:MicroTechnology-Services/microtechnology-services.github.io.git mtsi-blog
    cd mtsi-blog
    docker run -it --rm --label=jekyll --volume=$(pwd):/srv/jekyll -p 127.0.0.1:4000:4000 jekyll/jekyll

Then visit http://localhost:4000 You can add posts in `_posts`. More editing details available in the [Jekyll Docs][jekyll].

[jekyll]: https://jekyllrb.com/docs/home/
