# [Ghost](https://github.com/TryGhost/Ghost) on [Dokku](http://dokku.viewdocs.io/dokku/)

Ghost is a free, open, simple blogging platform. Visit the project's website at <http://ghost.org>, or read the docs on <http://support.ghost.org>.

Thanks project is based on cobysim [ghost-on-heroku ](https://github.com/cobyism/ghost-on-heroku) repo.

## Ghost version 1.X

The latest release of Ghost is supported!

  * Requires MySQL database, available through mariadb plugin and Mailgun account:
    * [dokku mariadb](https://github.com/dokku/dokku-mariadb)
    * [mailgun](https://www.mailgun.com/)

### Things you should know

After deployment,
- First, visit Ghost at `https://YOURAPPNAME.dokku-domain.com/ghost` to set up your admin account
- The app may take a few minutes to come to life
- Your blog will be publicly accessible at `https://YOURAPPNAME.dokku-domain.com`
- If you subsequently set up a custom domain for your blog, you’ll need to update your Ghost blog’s `PUBLIC_URL` environment variable accordingly
- If you create much content or decide to scale-up web process to support more traffic.

#### Configuring S3 file uploads

To configure S3 file storage, create an S3 bucket on Amazon AWS, and then specify the following details as environment variables on the Dokku deployment page:

- `S3_ACCESS_KEY_ID` and `S3_ACCESS_SECRET_KEY`: **Required if using S3 uploads**. These fields are the AWS key/secret pair needed to authenticate with Amazon S3. You must have granted this keypair sufficient permissions on the S3 bucket in question in order for S3 uploads to work.

- `S3_BUCKET_NAME`: **Required if using S3 uploads**. This is the name you gave to your S3 bucket.

- `S3_BUCKET_REGION`: **Required if using S3 uploads**. Specify the region the bucket has been created in, using slug format (e.g. `us-east-1`, `eu-west-1`). A full list of S3 regions is [available here](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region).

- `S3_ASSET_HOST_URL`: Optional, even if using S3 uploads. Use this variable to specify the S3 bucket URL in virtual host style, path style or using a custom domain. You should also include a trailing slash (example `https://my.custom.domain/`).  See [this page](http://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html) for details.

Once your app is up and running with these variables in place, you should be able to upload images via the Ghost interface and they’ll be stored in Amazon S3. :sparkles:

### How this works

This repository is a [Node.js](https://nodejs.org) web application that specifies [Ghost as a dependency](https://docs.ghost.org/v1.0.0/docs/using-ghost-as-an-npm-module), and makes a deploy button available.

  * Ghost and Casper theme versions are declared in the Node app's [`package.json`](package.json)
  * Scales across processor cores in larger dynos via [Node cluster API](https://nodejs.org/dist/latest-v6.x/docs/api/cluster.html)

## Deploy app

To deploy this dokku app you need to first create the app in your dokku server. We will not cover installing dokku or plugins. You can clone (or a fork) f this repo.

```bash
dokku apps:create ghost-blog
```

We need to create the database and link it to our app to set the DATABASE_URL enviroment variable for our ghost-blog.

```bash
dokku mariadb:create ghost-blog-bd
dokku mariadb:link ghost-blog-bd ghost-blog
```

Not its time to config some enviroment varaibles for Mailgun (so you can recover your account via emial) and S3 (upload images) to get the app working.
```bash
dokku config:set ghost-blog MAILGUN_SMTP_LOGIN=GET_FROM_MAILGUN NODE_ENV=production MAILGUN_SMTP_PASSWORD=GET_FROM_MAILGUN MAILGUN_SMTP_PORT=587 MAILGUN_SMTP_SERVER=smtp.mailgun.org S3_ACCESS_KEY_ID=GET_FROM_S3 S3_ACCESS_SECRET_KEY=GET_FROM_S3 S3_BUCKET_NAME=GET_FROM_S3 S3_BUCKET_REGION=GET_FROM_S3 S3_BUCKET_REGION=GET_FROM_S3
```

Now its time for deploy your code!
```bash
git clone https://github.com/JNajera/ghost-on-dokku
cd ghost-on-dokku

git add remote dokku dokku@dokku-domain.com:ghost-blog
git push dokku master
```

### Upgrading Ghost

On each deployment, the Dokku Node/npm build process will **auto-upgrade Ghost to the newest 1.x version**. To prevent this behavior, use npm 5+ (or yarn) to create a lockfile.

```bash
npm install
git add package-lock.json
git commit -m 'Lock dependencies'
git push dokku master
```

Now, future deployments will always use the same set of dependencies.

To update to newer versions:

```
npm update
git add package-lock.json
git commit -m 'Update dependencies'
git push dokku master
```

### Database migrations

Newer versions of Ghost frequently require changes to the database. These changes are automated with a process called **database migrations**.

After upgrading Ghost, you may see errors logged like:

> DatabaseIsNotOkError: Migrations are missing. Please run knex-migrator migrate.

To resolve this error, run the pending migrations and restart to get the app back on-line:

```bash
dokku run dokku-app-name knex-migrator migrate --mgpath node_modules/ghost
dokku ps:restart dokku-app-name
```

This can be automated by adding the following line to `Procfile`:

```
release: knex-migrator migrate --mgpath node_modules/ghost
```

## License

Released under the [MIT license](./LICENSE), just like the Ghost project itself.
