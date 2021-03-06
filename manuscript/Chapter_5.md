# Chapter 5: The Testing Framework Part 2 (Optional)

This chapter talks about [Codeception](http://codeception.com/). Feel free to skip it if you already have a testing framework in place.

Since we are ready to build the application, let us remove the route for the default homepage and we are going to make sure that we have done that correctly.

## Modifying DefaultController.php

Previously, we could access the route "/" because the route exists in DefaultController.php. Removing the @route annotation will remove the route. A simple trick to do that is to take out the @.

```
#  symfony/src/AppBundle/Controller/DefaultController.php

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Request;

class DefaultController extends Controller
{
    /**
     * not using the homepage route for now
     *
     * Route("/", name="homepage")
     */
    public function indexAction(Request $request)
    {
        // replace this example code with whatever you need
        return $this->render('default/index.html.twig', array(
            'base_dir' => realpath($this->container->getParameter('kernel.root_dir').'/..'),
        ));
    }
}
```

Now, refresh http://songbird.app:8000/app_dev.php and you should see a 404 error.

```
No route found for "GET /"
```

This is correct because the url is no longer configured. How can you be sure? Let us check it out from the command line

```
# in symfony
-> docker-compose exec php bin/console debug:router

[router] Current routes
 Name                     Method Scheme Host Path
 _wdt                     ANY    ANY    ANY  /_wdt/{token}
 _profiler_home           ANY    ANY    ANY  /_profiler/
 _profiler_search         ANY    ANY    ANY  /_profiler/search
 _profiler_search_bar     ANY    ANY    ANY  /_profiler/search_bar
 _profiler_purge          ANY    ANY    ANY  /_profiler/purge
 _profiler_info           ANY    ANY    ANY  /_profiler/info/{about}
 _profiler_phpinfo        ANY    ANY    ANY  /_profiler/phpinfo
 _profiler_search_results ANY    ANY    ANY  /_profiler/{token}/search/results
 _profiler                ANY    ANY    ANY  /_profiler/{token}
 _profiler_router         ANY    ANY    ANY  /_profiler/{token}/router
 _profiler_exception      ANY    ANY    ANY  /_profiler/{token}/exception
 _profiler_exception_css  ANY    ANY    ANY  /_profiler/{token}/exception.css
 _configurator_home       ANY    ANY    ANY  /_configurator/
 _configurator_step       ANY    ANY    ANY  /_configurator/step/{index}
 _configurator_final      ANY    ANY    ANY  /_configurator/final
 _twig_error_test         ANY    ANY    ANY  /_error/{code}.{_format}
```

Looks like there is no trace of the / path. This is all good but to do a proper job, we have to make sure that this logic is remembered in the future. We need to record this logic in a test.

Hang on, did you realise how ugly the command line is? 

`docker-compose exec php bin/console debug:router`

What we are doing here is that we are trying to access `bin/console` within the php docker instance. If you have php 7 installed in your host, you could run `bin/console` straight away and achieve the same results.

We will create a much simple wrapper around the docker commands in a minute.

## Making sure the / route is removed

To make sure that / route is correctly removed and not accidentally added again in the future, let us add a test for it.


```
# symfony/tests/acceptance/AppBundleCest.php
...
# replace installationTest with removalTest

   /**
     * check that homepage is not active
     *
     * @param AcceptanceTester $I
     */
    public function RemovalTest(AcceptanceTester $I)
    {
        $I->wantTo('Check if / is not active.');
        $I->amOnPage('/');
        $I->see('404 Not Found');
    }
```

and run the test again,

```
# clear prod cache because test is running in prod env
-> docker-compose exec php bin/console cache:clear --env=prod

# remember to start phantomjs server before running this command
-> docker-compose exec php vendor/bin/codecept run acceptance
...
Time: 10.47 seconds, Memory: 11.50MB

OK (1 test, 1 assertion)
```

## Creating custom bash script to run acceptance test

We have to remember to clear the cache every time we run the test so that we don't test on the cached version. Let us automate this by creating a script in the scripts dir called "runtest" and make it executable.

```
# in symfony
-> touch scripts/runtest
-> chmod u+x scripts/runtest
```

In the runtest script,

```
# symfony/scripts/runtest

#!/bin/bash

docker-compose exec php bin/console cache:clear --no-warmup
docker-compose exec php vendor/bin/codecept run acceptance
```

Now test your automation by running

```
-> ./scripts/runtest
...
OK (1 test, 1 assertion)
```

We are almost done. Remember to commit all your changes before moving on to the next chapter.

## Summary

In this chapter, we have removed the default / route and updated our test criteria. We have also created a few bash scripts to automate the task of running codecept test. We will add more to these scripts in the future.

## References

* [TDD](https://en.wikipedia.org/wiki/Test-driven_development)

* [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development)

* [PhantomJS](http://phantomjs.org/download.html)