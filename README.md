# fbpac-backend

This is the Rust backend (website, ad receiver) for the [Facebook Political Ad Collector](https://github.com/globeandmail/facebook-political-ads/). For a full breakdown of the other services you'll need to deploy the app, see the [README for our main repo](https://github.com/globeandmail/facebook-political-ads/blob/master/README.md).


### Installation

There's two ways of running this: the easy, Docker way, or the hard way.


##### The Docker way

For development and evaluation purposes, you can easily set up the FBPAC backend environment using Docker. You'll need to clone both this repo and the [fbpac-api](https://github.com/globeandmail/fbpac-api) repos, then run `docker-compose -f docker-compose/docker-compose.yml up` in the root of this directory.

```sh
cd ..
git clone https://github.com/globeandmail/fbpac-api
cd fbpac-api
bundle install
cd ../fbpac-backend
docker build .
```

(You may need to pass an HTTP_PROXY environment variable as per [this StackOverflow issue](https://stackoverflow.com/questions/34302974/how-could-i-execute-apt-get-install-on-docker-ubuntu-contain).)

Now visit `localhost:8080`; you should see the dashboard.


##### The hard way

This runs Cargo and Diesel manually.

First, make sure you have Rust [installed](https://doc.rust-lang.org/cargo/getting-started/installation.html):

```sh
curl -sSf https://static.rust-lang.org/rustup.sh | sh
```

Be sure to add `~/.cargo/bin` (or wherever cargo is installed, `path/to/.cargo/bin`) to your PATH. You can do this by adding this line to your `.bash_rc` or `.zshrc` or whatever config file you typically use for your shell.

```sh
PATH=$PATH:~/.cargo/bin
```

The backend server is a Rust application that runs on top of [diesel](https://diesel.rs) and [hyper](https://hyper.rs/). You'll need the diesel command line interface to get started and to create the database:

```sh
cargo install diesel_cli
diesel database setup
```

You can kick the tires by running:

```sh
cd server
cargo build
cargo run
```

This will give a server running at `localhost:8080`. You will also need to build the backend's static resources. To do this, in another terminal tab:

```sh
cd server/public
npm install
NODE_ENV=development npm run watch
```

This will build the required static assets (JavaScript and CSS) to view the admin console at `localhost:8080/facebook-ads/`.

If you make any changes to the database, after you run the migration, you'll want to update the schema with `diesel print-schema > src/schema.rs`. You'll need to do this even if you make changes via a Rails migration.

The backend has both unit and integration tests. You will need to set up a test database alongside your actual database in order to run the integration tests. To do this, you will need to the same as above, but substitute out the database test URL:

```sh
diesel database setup --database-url postgres://localhost/facebook_ads_test
```

Note that the value for `--database-url` came from the `TEST_DATABASE_URL` value set in the `.env` file. Make sure that the urls match before you run the tests!

Additionally, because the integration tests use the database, we want to make sure that they aren't run in parallel, so to run the tests:

```sh
RUST_TEST_THREADS=1 cargo test
```

This will run the tests in sequence, avoiding parallelism errors for tests that require the database.


### Deployment
