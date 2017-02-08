## Personal blog

This site uses the HMFaysal Omega theme.

Ensure Ruby is installed, and install the required gems:

`gem install bundler`

`bundler install`

There are a few things you can do:

* `jekyll serve` to test locally.
* `jekyll build` to populate the _"_site"_ directory.
* `s3_website cfg create` to generate an S3 config file skeleton
* `s3_website cfg apply` sets up the S3 bucket to host a static website.
* `s3_website push` pushes the site you compiled earlier to S3
