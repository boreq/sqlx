# sqlx

A fork of [sqlx](https://github.com/jmoiron/sqlx) which fixes the way structs
are populated.

>Because of the way that sqlx builds the mapping of field name to field address,
>by the time you Scan into a struct, it no longer knows whether or not a name
>was encountered twice during its traversal of the struct tree. So unlike Go,
>StructScan will choose the "first" field encountered which has that name.

This fork fixes this very annoying behaviour (which in my opinion makes the
entire library broken). It is now possible to do the following, despite the
fact that both structs have duplicated fields called `ID` and `Title`:

    type Blog struct {
        ID    uint
        Title string
    }

    type Post struct {
        ID     uint
        BlogID uint
        Title  string
    }

    type PostAndBlog {
        Post
        Blog
    }

    var posts []PostAndBlog
    if err := database.DB.Select(&posts,
        `SELECT post.*, blog.*
         FROM post
         JOIN blog ON blog.id = post.blog_id`); err != nil {
         panic(err)
     }

There is only a single caveat: please remember to embed structs in the same
order in which they are selected in the database query. As you can see in the
above example `Blog` is embedded in the struct after `Post` and it is also
selected second in the SQL query. It is possible to embed any number of
structs.

## Disclaimer
This fix was created quickly and without much though put into it, it is
possible that it breaks other features of the library without my knowledge.
