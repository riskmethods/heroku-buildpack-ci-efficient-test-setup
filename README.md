# The problem

If you have the following setup:

  * a monolithic application on Heroku meaning that you have to run frontend and backend tests together on single CI build
  * running backend tests on parallel on multiple Heroku dynos.

Then this setup is not efficient for the dynos which run only backend tests because Heroku test setup
still builds and compiles JS dependencies using Node JS buildpack.

Unfortunately, Heroku's `app.json` file setup does not allow us to conditionally specify which buildpacks to run.
In such case we would just have disabled Node JS buildpack for such dynos.

# The solution

Detect that the current dyno is not going to run JS tests with help of `CI_NODE_INDEX` env var.
In this case we make the `package.json` file in the project directory empty so that Node JS buildpack
quickly finishes its job (without installing a lot of JS dependencies).

# The usage

Add line `{ "url":  "https://github.com/riskmethods/heroku-buildpack-ci-efficient-test-setup" }` to `app.json`
before the nodejs buildpack. Example:

```json
 "environments": {
    "test": {
      "buildpacks": [
        { "url":  "https://github.com/riskmethods/heroku-buildpack-ci-efficient-test-setup" },
        { "url": "heroku/nodejs"},
        { "url": "heroku/ruby" }
      ]
    }
  }
```

Also, pass `JS_TESTS_NODE_INDEX` env var to your CI build (using `app.json` or via pipeline settings).
This variable defines on which node the JS tests are run.
