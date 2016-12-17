## Personal blog

This site uses the HMFaysal Omega theme.

`jekyll serve` to test locally.

`jekyll build` to populate the _"_site"_ directory.

Ensure Ruby is installed, and install the required gems:

`gem install bundler`

`bundler install`

Generate an S3 config file:

`s3_website cfg create`

Fill in the s3_website.yml config file with your secrets

`s3_website cfg apply` sets up the S3 bucket to host a static website.

`s3_website push` pushes the website to S3
