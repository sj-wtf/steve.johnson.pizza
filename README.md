# HMFAYSAL OMEGA THEME

`jekyll serve` to test locally.
`jekyll build` to populate the _site directory.

Ensure Ruby is installed

`gem install bundler`
`bundler install`
`s3_website cfg create`

Fill in the s3_website.yml config file with your secrets

`s3_website cfg apply` sets up the S3 bucket to host a static website.

`s3_website push` pushes the website to S3
