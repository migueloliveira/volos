# volos-quota-common

This module adds support for "quotas" to any API.

A quota is a traffic-management construct that is often used in building APIs and presenting them to
developers. It is common in the API world to offer a service that is available to each individual application
for a limited number of requests per hour, day, or week.

For instance, the developers of an API may wish to identify each application that uses the API using an "API key"
and offer different levels of service  based on the application. For instance, some applications might be allowed
to make only 100 API calls per hour, while others might have a much higher limit.

This module may be used to do that. Using it, you create a "quota," and each quota has the following attributes:

* timeUnit: How often the quota resets -- may be in minutes, hours, days, or weeks
* interval: Works with the timeUnit to determine how often the quota resets. For instance, every 5 days or 2 weeks.
* startTime: A time at which the quota calculations should begin. For instance, if there is no start time then a
quota set to reset in "one day" will reset 24 hours after the first message is receiver, but if the start time
is set to the top of the hour on some day, then the quota will always reset at the top of the hour.

Once a quota has been created, you "apply" the quota, which involves setting some additional attributes:

* weight: How much to add to the quota -- the default is 1, but in some advanced cases, API providers will
assign different weights to different API calls.
* allow: How many requests to allow. By specifying this here, rather than when the quota is created,
we can have different quotas for different apps.
* identifier: Identifies the quota bucket. This is a string that may be set to any value. Each identifier locates
a single quota bucket, which has a separate counter value from other counters.

## Modules

This module does not do anything on its own -- it depends on an implementation which stores the quota in a particular
place. Currently the options are:

* volos-quota-local: Stores quotas in memory on the node where the script is running
* volos-quota-apigee: Communicates with Apigee via API to update and check quota results.

## Example

    var apiKey = // Some string that comes from the incoming API call
    var qm = require('volos-quota-apigee');
    var quota = qm.createQuota({ timeUnit: 'day', interval: 1 });
    // Allow 100 requests per day for each application, based on API key
    quota.apply({ identifier: apiKey, allow: 100 });

## Reference

### Class: Quota

### Quota.apply(options, callback)

Apply the quota and invoke "callback" with a result. Options can have the following parameters:

* identifier (required): A string that identifies the quota bucket. Quotas with different identifiers
are treated as separate quotas.
* weight (optional): A number that indicates how much to increment the quota counter by. Default is 1.
* allow (optional): How much to allow this quota before it is used up. If this is not specified here,
then the value of "allow" that was specified when the quota object was created is used.

The result of this call is delivered by calling "callback." If there was an error, then the first parameter
will be an Error object. Otherwise, the first parameter will be "undefined" and the second will be
an object that contains the following fields:

* allowed: (number) How much is allowed in the quota bucket. Essentially the same as the "allow" parameter that is
passed to the "apply" method, or the default that came from the overall object.
* used: (number) How much of the quota bucket is used up.
* isAllowed: (boolean) Whether the quota bucket has been used up. Basically the same as checking whether
"used > allowed".


