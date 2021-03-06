---
title: New Relic Performance Analysis on Pantheon
description: Learn how to utilize New Relic performance metrics and reports for your Pantheon site.
categories: [developing]
tags: [debug, featured]
keywords: new relic, new relic pro, performance analysis, enable new relic, what is new relic, add new relic, mysql performance, performance, authenticated users, how to log authenticated users, how to use new relic, using new relic, sql performance
---
New Relic offers a wide array of metrics that provide a nearly real-time look into the performance of a web application.

Enabling New Relic on Pantheon not only makes it easy for you to monitor to your performance, but it can also speed-up the support process by helping our support team visualize corresponding performance and symptoms.

## Enable New Relic

To enable New Relic on your Pantheon site, click **Settings** in the upper-right corner of your Site Dashboard. Within the Add Ons tab, click **Add** next to New Relic.

![Pantheon Dashboard enable New Relic](/source/docs/assets/images/new-relic-add-on-image.png)  
Pantheon will automatically configure New Relic on your behalf, including all configurations on the server.

Access New Relic by clicking selecting the **New Relic** tab on the Site Dashboard.  

You should visit your site in the browser a couple of times to generate data in New Relic. After a few minutes pass, go to the New Relic workspace on your Dashboard, and click **Open New Relic**.

The New Relic interface provides severals views that display information about various aspects of your website's performance.

## End-User Overview

![End-User Interface](/source/docs/assets/images/new-relic-end-user.png)

1. The **Browser Page Load Time** represents the average time it takes the browser to process and render the page once Nginx has sent out the data. 
2. The **Apdex Score** attempts to break down the end-user response times into three categories based upon load-time thresholds.
<div class="alert alert-info" role="alert">
<h4>Note</h4>
The apdex is not the most accurate representation of your sites' load-times. It is simply there to give you a broad idea. The provided slow traces are the key to figuring out why your site is running poorly.</div>
3. **Page Views** provides the number of page views per minute.
4. The **Apdex & Top States by Load Time** presents a visualization of load times based on region; these can be viewed internally to the United States or globally.
5. **Recent Events** logs the most recent events.

## Appserver Overview

![Appserver Overview](/source/docs/assets/images/new-relic-overview.png)

1. The **response time** represents the duration that it takes for the request to be received by Nginx, processed by the application server, and returned to the client for processing and rendering. The end-user response time is different than the appserver response time, as the end user's response time is affected by variables such as connection speed, network latency, and browser.
2. The **slow transactions** represent points of the process that take longer than a defined amount of time. This feature can be used to test a site against a set of timed-expectations to discover and fix problems.
3. **Errors** captures and logs any error generated by the application stack.
4. **Throughput** displays the number of requests per minute.
5. The **Servers** display shows the number of [application containers](/docs/application-containers), or appservers, associated with that environment.

## Only Log Authenticated Users

If your site consists of mostly authenticated traffic, it can be useful to exclude anonymous users who are using your site's page cache. This technique will still capture form submissions, including logins and contact pages. Similar logic can be used to disable New Relic on certain paths, such as `/admin` in Drupal or `/wp-admin` in WordPress.  

### Drupal
To disable New Relic for anonymous traffic on Drupal-based sites, add the following to your `sites/default/settings.php`:

```
// Disable New Relic for anonymous users.
if (function_exists('newrelic_ignore_transaction')) {
  $skip_new_relic = TRUE;
  // Capture all transactions for users with a PHP session.
  foreach (array_keys($_COOKIE) as $cookie) {
    if (substr($cookie, 0, 4) == 'SESS') {
      $skip_new_relic = FALSE;
    }
  }
  // Capture all POST requests so we include anonymous form submissions.
  if (isset($_SERVER['REQUEST_METHOD']) && $_SERVER['REQUEST_METHOD'] == 'POST') {
    $skip_new_relic = FALSE;
  }
  if ($skip_new_relic) {
    newrelic_ignore_transaction();
  }
}
```

### WordPress
To disable New Relic for anonymous traffic on WordPress sites, add the following to your `templates/<your_template>/functions.php`:

```
// Disable New Relic for anonymous users.
if (function_exists('newrelic_ignore_transaction')) {
    $skip_new_relic = !is_user_logged_in();

    // Capture all POST requests so we include anonymous form submissions.
    if (isset($_SERVER['REQUEST_METHOD']) &&
        $_SERVER['REQUEST_METHOD'] == 'POST') {
        $skip_new_relic = FALSE;
    }

    if ($skip_new_relic) {
        newrelic_ignore_transaction();
    }
}
```
## AMP Validation Errors
New Relic's Browser agent JavaScript tag may cause [Google AMP validator](https://www.ampproject.org/docs/guides/validate.html) failures, such as `The tag 'script' is disallowed except in specific forms`. You can resolve validation errors by disabling New Relic's Browser monitoring agent on all AMP pages.

Start by strategizing and testing logic to identify AMP pages based on your site's implementation. Once confirmed, place AMP isolating logic within the first conditional statement of the example below. Then disable New Relic's Browser monitoring by setting `newrelic_disable_autorum` to `FALSE` only if the current request is an AMP page:  

```php
if (extension_loaded('newrelic')) {
  // Add AMP page identifying logic here that would, for example,    
  // set variable $amp to TRUE or FALSE. If $amp is true, disable new relic
  if ($amp) {
    newrelic_disable_autorum (FALSE);
  }
}
```

## Frequently Asked Questions

#### How can I share a link to a particular metric?

In the New Relic performance page, click **Permalink**. This will preserve the current time window and take the link recipient to the same page you're currently looking at.  
 ![Permalink on the New Relic performance page](/source/docs/assets/images/new-relic-permalink.png)

#### How much is New Relic?

Pantheon provides New Relic Standard at no cost. You can upgrade your site monitoring to Professional or above by contacting New Relic through your New Relic interface, and they can provide a quote. Pantheon cannot provide a New Relic quote. Make sure they know you're a Pantheon customer for a discount!

#### How does New Relic price Pantheon sites for the Professional service?

New Relic sites on Pantheon are priced per Application Container, rather than by core. For example, a site with three environments with two application containers in the Live environment will be priced as four total application containers.

#### Will turning on New Relic slow my site down?

Basically no, New Relic will not make your site slower. There is a very small amount of overhead, but it's imperceptible. The amount of available metrics useful for debugging and improving performance far outstrips the negligible difference.

#### What is the difference between app server response time and browser page load time?

App server response time measures how the page was built on Pantheon, including PHP execution, database, Redis (if used). Browser page load time measures the additional time of client-side page rendering, DOM processing, and how long it took to transfer to the client. While a fast app server response time is optimal, a slow browser page load time indicates a bad user experience. Some causes are unaggregated or uncompressed scripts and stylesheets, invalid markup, or unoptimized client-side code (like JavaScript).

#### Can I use my existing New Relic license with my Pantheon site?

New Relic is automatically provisioned for your site. Unfortunately, you cannot use your existing license.

#### Why are servers listed in New Relic with no data?

Because Pantheon's runtime matrix runs your application across many containers simultaneously, it's common to see old containers with no reporting data as your application shifts around. This is not a cause for concern.

## Resources

- [General New Relic Documentation](https://newrelic.com/docs/)
- [Official New Relic Videos and Tutorials](http://newrelic.com/resources/videos)
- [Case Studies](http://newrelic.com/resources/case-studies)
- [What is Real User Monitoring?](https://newrelic.com/docs/features/real-user-monitoring)
- [Finding Help From the New Relic UI](https://newrelic.com/docs/site/finding-help)
- [Interface Overview](https://newrelic.com/docs/site/the-new-relic-ui)
