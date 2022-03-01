# Learning objectives
After completing this module, you'll be able to:

- Identify scenarios for which autoscaling is an appropriate solution.
- Create autoscaling rules for a web app.
- Monitor the effects of autoscaling.

## Autoscaling
Autoscaling enables a system to adjust the resources required to meet the varying demand from users, while controlling the costs associated with these resources. Autoscaling requires you to configure autoscale rules that specify the conditions under which resources should be added or removed.

# Examine autoscale factors
Autoscaling can be triggered according to a schedule, or by assessing whether the system is running short on resources.

Autoscaling performs scaling in and out, as opposed to scaling up and down.

## Azure App Service Autoscaling

Autoscaling in Azure App Service monitors the resource metrics of a web app as it runs. It detects situations where additional resources are required to handle an increasing workload, and ensures those resources are available before the system becomes overloaded.

## Autoscaling rules

A rule specifies the threshold for a metric, and triggers an autoscale event when this threshold is crossed. Autoscaling can also deallocate resources when the workload has diminished.

Define your autoscaling rules carefully. For example, a Denial of Service attack will likely result in a large-scale influx of incoming traffic. Trying to handle a surge in requests caused by a DoS attack would be fruitless and expensive. These requests aren't genuine, and should be discarded rather than processed. A better solution is to implement detection and filtering of requests that occur during such an attack before they reach your service.

## When should you consider autoscaling?
it's a suitable solution when hosting any application when you can't easily predict the workload in advance, or when the workload is likely to vary by date or time. For example, you might expect increased/reduced activity for a business app during holidays.

**Autoscaling works by adding or removing web servers**
if your web apps perform resource-intensive processing as part of each request, then autoscaling might not be an effective approach. In these situations, manually scaling up may be necessary.

Autoscaling isn't the best approach to handling long-term growth. You might have a web app that starts with a small number of users, but increases in popularity over time. Autoscaling has an overhead associated with monitoring resources and determining whether to trigger a scaling event. In this scenario, if you can anticipate the rate of growth, manually scaling the system over time may be a more cost effective approach.

The number of instances of a service is also a factor. You might expect to run only a few instances of a service most of the time. However, in this situation, your service will always be susceptible to downtime or lack of availability whether autoscaling is enabled or not.

# Identify autoscale factors

## Autoscaling and the App Service Plan
To prevent runaway autoscaling, an App Service Plan has an instance limit. Plans in more expensive pricing tiers have a higher limit. Autoscaling cannot create more instances than this limit.
**Not all App Service Plan pricing tiers support autoscaling.**

## Autoscale conditions
Azure provides two options for autoscaling:

- Scale based on a metric, such as the length of the disk queue, or the number of HTTP requests awaiting processing.
- Scale to a specific instance count according to a schedule. For example, you can arrange to scale out at a particular time of day, or on a specific date or day of the week. You also specify an end date, and the system will scale back in at this time.

Scaling to a specific instance count only enables you to scale out to a defined number of instances. If you need to scale out incrementally, you can combine metric and schedule-based autoscaling in the same autoscale condition.

An App Service Plan also has a default condition that will be used if none of the other conditions are applicable. This condition is always active and doesn't have a schedule.

## Metrics for autoscale rules
- **CPU Percentage**. This metric is an indication of the CPU utilization across all instances. A high value shows that instances are becoming CPU-bound, which could cause delays in processing client requests.
- **Memory Percentage**. This metric captures the memory occupancy of the application across all instances. A high value indicates that free memory could be running low, and could cause one or more instances to fail.
- **Disk Queue Length**. This metric is a measure of the number of outstanding I/O requests across all instances. A high value means that disk contention could be occurring.
- **Http Queue Length**. This metric shows how many client requests are waiting for processing by the web app. If this number is large, client requests might fail with HTTP 408 (Timeout) errors.
- **Data In**. This metric is the number of bytes received across all instances.
- **Data Out**. This metric is the number of bytes sent by all instances.

## How an autoscale rule analyzes metrics

Analysis is a multi-step process:

1. In the first step, an autoscale rule aggregates the values retrieved for a metric for all instances across a period of time known as the **time grain**.  Each metric has its own time grain, in most cases this period is **1 minute**  The aggregated value is known as the **time aggregation**. The options available are **Average**, **Minimum**, **Maximum**, **Total**, **Last**, and **Count**.

2. An autoscale rule that performs a further aggregation of the value calculated by the time aggregation over a longer, **user-specified period**, known as the **Duration**.
 - The minimum Duration is `5 minutes` (If the Duration is set to 10 minutes for example, the autoscale rule will aggregate the 10 values calculated for the time grain.)
3.  If the time grain statistic is set to Maximum, and the Duration of the rule is set to 10 minutes, the maximum of the 10 average values for the CPU percentage utilization will be used to determine whether the rule threshold has been crossed.
  

## Autoscale actions
When a metric has crossed a threshold an Autoscale action is performed.
An autoscale action uses an operator `(>, <, == , etc)` to determine how to react to the threshold.

An autoscale action can be:
- **scale-out**: increases the number of instances. Typically use the greater than operator to compare the metric value to the threshold.
- **scale-in**:  reduces the instance count. Tend to compare the metric value to the threshold with the less than operator
 
**An autoscale action can also set the instance count to a specific level, rather than incrementing or decrementing the number available.**

An autoscale action has a **cool down period**, specified in minutes. During this interval, the scale rule won't be triggered again. The minimum cool down period is **five minutes**.

## Pairing autoscale rules
Plan your rules:
- One autoscale rule should indicate how to scale the system out when a metric exceeds an upper threshold. 
- Then other rule should define how to scale the system back in again when the same metric drops below a lower threshold.

## Combining autoscale rules

A single autoscale condition can contain several autoscale rules.

However, the autoscale rules in an autoscale condition don't have to be directly related. You could define the following four rules in the same autoscale condition:

- If the HTTP queue length exceeds 10, scale out by 1
- If the CPU utilization exceeds 70%, scale out by 1
- If the HTTP queue length is zero, scale in by 1
- If the CPU utilization drops below 50%, scale in by 1

# Enable autoscale in App Service
Not all tiers support autoscaling:
- The development pricing tiers are either limited to a single instance(the F1 and D1 tiers
- or they only provide manual scaling (the B1 tier_
- if you've selected one of these tiers, you must first scale up to the S1 or any of the P level production tiers.

1. Go to your App Service plan in the Azure portal and select Scale out (App Service plan) in the Settings group in the left navigation pane.

By default, an App Service Plan only implements manual scaling. Selecting Custom autoscale reveals condition groups you can use to manage your scale settings.

## Add scale conditions
Once you enable autoscaling, you can edit the automatically created default scale condition, and you can add your own custom scale conditions
The Default scale condition is executed when none of the other scale conditions are active.

A metric-based scale condition can also specify the minimum and maximum number of instances to create. The maximum number can't exceed the limits defined by the pricing tier.

## Create scale rules
You use the **Add a rule link** to add your own custom rules. You define the criteria that indicate when a rule should trigger an autoscale action, and the autoscale action to be performed (scale out or scale in) using the metrics, aggregations, operators, and thresholds described earlier.

## Monitor autoscaling activity

The Azure portal enables you to track when autoscaling has occurred through the Run **history chart**
You can use the Run history chart in conjunction with the metrics shown on the Overview page to correlate the autoscaling events with resource utilization.

# Explore autoscale best practices

## Autoscale concepts
- An autoscale setting scales instances horizontally, which is out by increasing the instances and in by decreasing the number of instances. An autoscale setting has a maximum, minimum, and default value of instances.

- An autoscale job always reads the associated metric to scale by, checking if it has crossed the configured threshold for scale-out or scale-in. 

- All thresholds are calculated at an instance level. For example, "scale out by one instance when average CPU > 80% when instance count is 2".

- All autoscale successes and failures are logged to the Activity Log. You can then configure an activity log alert so that you can be notified via email, SMS, or webhooks whenever there is activity.

## Autoscale best practices
- Ensure the maximum and minimum values are different and have an adequate margin between them (If you have a setting that has minimum=2, maximum=2 and the current instance count is 2, no scale action can occur. Keep an adequate margin between the maximum and minimum instance counts, which are inclusive. Autoscale always scales between these limits.)
- Choose the appropriate statistic for your diagnostics metric (Average, Minimum, Maximum and Total as a metric to scale by). The most common is average
- Choose the thresholds carefully for all metric types:
  Estimation during a scale-in is intended to avoid "flapping" situations, where scale-in and scale-out actions continually go back and forth. Keep this behavior in mind when you choose the same thresholds for scale-out and in.

  We recommend choosing an adequate margin between the scale-out and in thresholds. As an example, consider the following better rule combination.

  Increase instances by 1 count when CPU% >= 80
  Decrease instances by 1 count when CPU% <= 60

## Considerations for scaling when multiple rules are configured in a profile

- On scale-out, autoscale runs if any rule is met.
- On scale-in, autoscale require all rules to be met.
## Always select a safe default instance count

## Configure autoscale notifications

Autoscale will post to the Activity Log if any of the following conditions occur:

- Autoscale issues a scale operation
- Autoscale service successfully completes a scale action
- Autoscale service fails to take a scale action.
- Metrics are not available for autoscale service to make a scale decision.
- Metrics are available (recovery) again to make a scale decision.

# Knowledge check
1. Which of these statements best describes autoscaling?
  Autoscaling is a scale out/scale in solution.
2. Which of these scenarios is a suitable candidate for autoscaling? 
  The number of users requiring access to an application varies according to a regular schedule. For example, more users use the system on a Friday than other days of the week.
3. There are multiple rules in an autoscale profile. Which of the following scale operations will run if any of the rule conditions are met?

  scale-out